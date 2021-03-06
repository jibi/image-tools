# syntax=docker/dockerfile:1.1-experimental

# Copyright 2020 Authors of Cilium
# SPDX-License-Identifier: Apache-2.0

ARG COMPILERS_IMAGE=docker.io/cilium/image-compilers:797061dcd0b87a238a26f135ec7c425a7ef0dda6@sha256:54862b2d2fb286a61d29587e941c4723ac953a0a4843f096ea7a19fdf0b485f9
ARG UBUNTU_IMAGE=docker.io/library/ubuntu:20.04@sha256:8bce67040cd0ae39e0beb55bcb976a824d9966d2ac8d2e4bf6119b45505cee64
ARG TESTER_IMAGE=docker.io/cilium/image-tester:f31e8b568511be3333594051aa40abe7996a4ca1@sha256:b6c5f8d0aec315330c53a9ee6ae1243f87259ae80d6b31845beeee531cd1d4c0

FROM --platform=linux/amd64 ${COMPILERS_IMAGE} as builder

COPY checkout-iproute2.sh /tmp/checkout-iproute2.sh
RUN /tmp/checkout-iproute2.sh

COPY build-iproute2-native.sh /tmp/build-iproute2-native.sh
RUN /tmp/build-iproute2-native.sh

COPY build-iproute2-cross-aarch64.sh /tmp/build-iproute2-cross-aarch64.sh
RUN /tmp/build-iproute2-cross-aarch64.sh

FROM ${UBUNTU_IMAGE} as rootfs
ARG TARGETPLATFORM

ENV DEBIAN_FRONTEND noninteractive

# hadolint ignore=SC2215
RUN --mount=type=tmpfs,target=/var/cache/apt --mount=type=tmpfs,target=/var/lib/apt \
  apt-get update \
    && apt-get install -y --no-install-recommends libelf1 libmnl0 \
    && apt-get purge --auto-remove -y

COPY --from=builder /out/${TARGETPLATFORM}/bin /usr/local/bin
COPY test /test/iproute2

FROM ${TESTER_IMAGE} as test
COPY --from=rootfs / /
RUN /test/bin/cst -C /test/iproute2

FROM scratch
LABEL maintainer="maintainer@cilium.io"
COPY /empty /tmp/empty
COPY --from=rootfs / /
