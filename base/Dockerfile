# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
ARG tag
FROM adoptopenjdk/openjdk11:alpine AS base

FROM golang:1.14 AS cli-build

ENV CGO_ENABLED=0
ENV GO111MODULE=on

RUN mkdir /swctl
WORKDIR /tmp

RUN git clone https://github.com/apache/skywalking-cli.git
RUN cd skywalking-cli && go mod download
RUN cd skywalking-cli && make linux && mv bin/swctl-*-linux-amd64 /swctl

FROM base AS es6
RUN echo "This is the stage that build the es6 image"
ENV SKYWALKING_TGZ=apache-skywalking-apm-bin.tar.gz

FROM base AS es7
RUN echo "This is the stage that build the es7 image"
ENV SKYWALKING_TGZ=apache-skywalking-apm-bin-es7.tar.gz

FROM ${tag} AS build

ENV SKYWALKING_HOME=/skywalking
RUN mkdir -p "${SKYWALKING_HOME}"
ENV TMPDIR=/tmp/sw
RUN mkdir -p "${TMPDIR}"
WORKDIR $TMPDIR

COPY dist/${SKYWALKING_TGZ} ./
COPY --from=cli-build /swctl /swctl

RUN tar -xvf /tmp/sw/${SKYWALKING_TGZ} --strip-components=1 -C /skywalking
RUN rm -rf /skywalking/bin
RUN mkdir -p /skywalking/bin
RUN cp /swctl/* /skywalking/bin
RUN chmod +775 /skywalking/bin

FROM adoptopenjdk/openjdk11:alpine

LABEL maintainer="hanahmily@apache.org,tanjian@apache.org"

COPY --from=build /skywalking /skywalking

RUN apk add --no-cache \
    bash
