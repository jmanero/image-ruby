##
## Build the target ruby-version on public oraclelinux:8 images. 8-slim images
## use `microdnf` instead of `yum`/`dnf`.
##
ARG BUILD_IMAGE=oraclelinux:8
ARG EXPORT_IMAGE=oraclelinux:8-slim

FROM ${BUILD_IMAGE} AS build
SHELL ["/usr/bin/bash", "-c"]

## Install build dependencies
RUN dnf install -y gcc make tar bzip2 gdbm-devel libffi-devel openssl-devel readline-devel zlib-devel

## Target version parameter
ARG RUBY_VERSION=2.7.0
ARG RUBY_SRC_SHA256=7aa247a19622a803bdd29fdb28108de9798abe841254fe8ea82c31d125c6ab26

## Optional source parameters
ARG RUBY_SRC_ARTIFACT="ruby-${RUBY_VERSION}.tar.bz2"
ARG RUBY_SRC_INDEX="https://cache.ruby-lang.org/pub/ruby/index.txt"

## Optional build parameters
ARG RUBY_CONFIG_FLAGS="--prefix /opt/ruby --disable-install-doc --disable-jit-support --without-git"

RUN mkdir -p /build

## Resolve URL for Ruby source artifact
RUN curl -sS ${RUBY_SRC_INDEX} | fgrep ${RUBY_SRC_ARTIFACT} | awk '{ print $2; }' > /build/sources

## Fetch artifact and write SHA256 checksum to a file
RUN curl -sSL $(cat /build/sources) > /build/${RUBY_SRC_ARTIFACT}
RUN echo "${RUBY_SRC_SHA256}  /build/${RUBY_SRC_ARTIFACT}" >> /build/shasums

## Validate and extract source artifacts
RUN sha256sum -c /build/shasums

RUN tar -xf /build/${RUBY_SRC_ARTIFACT} -C /build

WORKDIR /build/ruby-${RUBY_VERSION}
RUN ./configure ${RUBY_CONFIG_FLAGS}
RUN make
RUN make install

##
## Export the build result on a clean image
##
FROM ${EXPORT_IMAGE}

## Install openssl shared libraries for the openssl Ruby module
RUN microdnf install openssl-libs && microdnf clean all

## Import build result
COPY --from=build /opt/ruby /opt/ruby

## Add ruby executables to path
ENV PATH="/opt/ruby/bin:${PATH}"
