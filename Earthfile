# Copyright 2021 Open Source Robotics Foundation, Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
VERSION 0.6

FROM ubuntu:jammy

repos-file:
  # Disable prompting during package installation
  ARG DEBIAN_FRONTEND=noninteractive

  # Set the locale
  RUN apt-get update && apt-get install -y locales
  RUN locale-gen en_US en_US.UTF-8
  RUN update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
  ENV LANG=en_US.UTF-8

  # Add the ROS 2 apt repository
  RUN apt-get install -y software-properties-common
  RUN add-apt-repository universe
  RUN apt-get update && apt-get install -y curl gnupg lsb-release
  RUN curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
  RUN echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null

  # Install repos file generator script requirements.
  RUN apt-get update && apt-get install -y python3-rosinstall-generator ruby python3-vcstool git
  WORKDIR /root
  COPY --dir scripts/ ./
  COPY excluded-pkgs.txt ./
  COPY spaceros-pkgs.txt ./
  COPY spaceros.repos ./

  # generate `ros2.repos` file with rosinstall script
  RUN --no-cache sh scripts/generate-repos.sh
  # merge `ros2.repos` file with `spaceros.repos` file
  RUN --no-cache ruby scripts/merge-repos.rb
  RUN mkdir src
  # clone all dependencies to export `.repos` file with exact tags / commit hashes
  # this way we can be certain that we always pull same versions of dependent packages
  RUN --no-cache vcs import src < ros2.repos
  RUN --no-cache vcs export --exact-with-tags --nested src > ros2.repos
  SAVE ARTIFACT ros2.repos AS LOCAL ros2.repos
