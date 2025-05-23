# Copyright 2023 Antaris, Inc.
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

FROM ubuntu:22.04 as shared_app_image
COPY images/app-shared/root-entrypoint.sh /entrypoint.sh
CMD /entrypoint.sh
RUN mkdir -p /opt/antaris/app /opt/antaris/logs


# This image layers on support for cpp-based payload applications
FROM shared_app_image as cpp_app_image

RUN apt update && apt install -y python3 python3-pip
RUN python3 -m pip install --upgrade pip

# This is required for GPIO and UART access using FTDI
RUN pip install pylibftdi
RUN apt install -y libftdi1
RUN python3 -m pip install pyserial
RUN python3 -m pip install azure-storage-file-share
RUN python3 -m pip install requests
RUN python3 -m pip install python-can

RUN mkdir /stage
COPY dist/satos-payload-sdk-cpp.deb /stage
RUN apt update && apt install -y /stage/satos-payload-sdk-cpp.deb

COPY dist/grpc-dev.deb /stage
RUN apt install -y pkg-config /stage/grpc-dev.deb

RUN rm -fr /stage


# This image adds the SDK agent dependencies
FROM cpp_app_image as cpp_app_image_with_sdk_agent

RUN mkdir -p /opt/antaris/sdk-agent

RUN mkdir /stage
COPY dist/satos-payload-sdk-agent.deb /stage
RUN apt update && apt install -y /stage/satos-payload-sdk-agent.deb
RUN rm -fr /stage

ENV RUN_SDK_AGENT=1

# assume if SDK agent is installed that we will need a workspace
RUN mkdir /workspace
WORKDIR /workspace
