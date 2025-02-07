ARG UBUNTU_VERSION=20.04
ARG NVIDIA_CUDA_VERSION=11.8.0

FROM nvidia/cuda:${NVIDIA_CUDA_VERSION}-devel-ubuntu${UBUNTU_VERSION} AS base

ARG ROS_DISTRO=noetic

ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8
ENV DEBIAN_FRONTEND=noninteractive
ENV TERM=xterm-256color

RUN apt-get update && apt-get install -y --no-install-recommends\
    ssh \
    git \
    curl \
    wget \
    lsb-release \
    terminator \
    build-essential \
    cmake \
    python3-pip \
    python3-flake8 \
    && apt-get clean && rm -rf /var/lib/apt/lists/*
    
RUN apt-get update && apt-get install -y \
    && sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list' \
    && curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | apt-key add - \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
    ros-${ROS_DISTRO}-ros-base \
    && rm -rf /var/lib/apt/lists/*

RUN pip3 install --no-cache-dir --upgrade pip\
    argcomplete \
    flake8-blind-except \
    flake8-builtins \
    flake8-comprehensions \
    flake8-deprecated \
    flake8-return \
    flake8-length \
    flake8-todo \
    flake8-quotes \
    black \
    mypy \
    prospector \
    pylint \
    numpy \
    opencv-python \
    pyquaternion \
    python-dateutil \
    matplotlib \
    pytransform3d \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

RUN apt-get update && apt-get install -y --no-install-recommends\
    pkg-config \
    python3-catkin-tools \
    python3-rosdep \
    python3-rosinstall-generator \
    python3-vcstool \
    python3-pykdl \
    ros-$ROS_DISTRO-rqt \
    ros-$ROS_DISTRO-rqt-common-plugins \
    ros-$ROS_DISTRO-rqt-robot-plugins \
    ros-$ROS_DISTRO-rqt-joint-trajectory-controller \   
    ros-$ROS_DISTRO-roslint \
    ros-$ROS_DISTRO-rqt-gui \
    ros-$ROS_DISTRO-rqt-gui-py \
    ros-$ROS_DISTRO-rqt-py-common \
    ros-$ROS_DISTRO-kdl-parser \
    ros-$ROS_DISTRO-urdf-parser-plugin \
    ros-$ROS_DISTRO-urdfdom-py \
    ros-$ROS_DISTRO-kdl-parser-py \
    ros-$ROS_DISTRO-ros-industrial-cmake-boilerplate \
    ros-$ROS_DISTRO-industrial-robot-client \
    ros-$ROS_DISTRO-plotjuggler-ros \
    ros-$ROS_DISTRO-joint-state-publisher \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

WORKDIR /
RUN mkdir -p dev_ws/src
WORKDIR /dev_ws/src

FROM base AS zed

WORKDIR /
RUN apt-get update && apt-get install -y --no-install-recommends\
    zstd \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

RUN wget -q -O ZED_SDK_Ubuntu20_cuda11.8_v4.2.5.zstd https://download.stereolabs.com/zedsdk/4.2/cu11/ubuntu20?_gl=1*g5lkdk*_gcl_au*NjQ0NTEwODkxLjE3Mzg4NTUzODY

RUN chmod +x ZED_SDK_Ubuntu20_cuda11.8_v4.2.5.zstd && \
    ./ZED_SDK_Ubuntu20_cuda11.8_v4.2.5.zstd -- silent runtime-only skip_tools skip_cuda

RUN apt-get update && apt-get install -y --no-install-recommends\
    ros-$ROS_DISTRO-diagnostic-updater \
    ros-$ROS_DISTRO-image-geometry \
    ros-$ROS_DISTRO-image-transport-plugins \
    ros-$ROS_DISTRO-camera-info-manager \
    ros-$ROS_DISTRO-xacro \
    ros-$ROS_DISTRO-rgbd-launch \
    ros-$ROS_DISTRO-joint-state-publisher \
    ros-$ROS_DISTRO-robot-state-publisher \
    ros-$ROS_DISTRO-rtabmap \
    ros-$ROS_DISTRO-rtabmap-ros \
    ros-$ROS_DISTRO-depthimage-to-laserscan \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

WORKDIR /dev_ws/src

RUN git clone --recursive https://github.com/stereolabs/zed-ros-wrapper.git


FROM zed AS open3d

RUN pip3 install --no-cache-dir --upgrade open3d --ignore-installed PyYAML


FROM open3d AS build

COPY . /dev_ws/src/

WORKDIR /dev_ws
RUN ["/bin/bash", "-c", "source /opt/ros/$ROS_DISTRO/setup.bash &&\
    catkin config --cmake-args -DCMAKE_BUILD_TYPE=Release &&\
    catkin build \
    "]


FROM build AS bash

RUN ["/bin/bash", "-c", "cp /dev_ws/src/.docker/setup.bash /dev_ws/setup.bash && chmod 777 /dev_ws/setup.bash"]
RUN ["/bin/bash", "-c", "cp /dev_ws/src/.docker/entrypoint.bash / && chmod 777 /entrypoint.bash"]
ENTRYPOINT ["bash", "/entrypoint.bash" ]

CMD ["bash"]
