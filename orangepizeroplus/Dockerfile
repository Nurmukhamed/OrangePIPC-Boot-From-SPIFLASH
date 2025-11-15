# Download base image ubuntu 20.04
FROM ubuntu:20.04 AS basebuild

# Disable Prompt During Packages Installation
ARG DEBIAN_FRONTEND=noninteractive
ARG GROUP=builder
ARG USER=builder

# Update Ubuntu Software repository
RUN apt-get update -qq -y &&\
    apt-get install -y \
      build-essential \
      git \
      libssl-dev \
      libc6-armel-cross \
      libc6-dev-armel-cross \
      binutils-arm-linux-gnueabi \
      libncurses5-dev \
      bison flex bc swig\
      gcc-arm-linux-gnueabihf g++-arm-linux-gnueabihf \
      gcc-arm-linux-gnueabi \
      g++-arm-linux-gnueabi \
      gcc-aarch64-linux-gnu \
      binutils-aarch64-linux-gnu \
      device-tree-compiler \
      curl wget \
      python3 python3-pip \
      plantuml \
      zip unzip &&\
   apt-get clean &&\
   rm -rf /var/lib/apt/lists/*

# Create group and user
RUN groupadd ${GROUP} &&\
    useradd -ms /bin/bash -g ${GROUP} ${USER} &&\
    mkdir /work &&\
    mkdir /output &&\
    chown ${USER}:${GROUP} /work /output &&\
    chmod 0755 /work &&\
    chmod 0777 /output

FROM basebuild AS armtrustedfirmware

WORKDIR /work

USER builder

RUN wget -q -c https://developer.arm.com/-/media/Files/downloads/gnu-a/8.3-2019.03/binrel/gcc-arm-8.3-2019.03-x86_64-aarch64-linux-gnu.tar.xz &&\
    tar xf gcc-arm-8.3-2019.03-x86_64-aarch64-linux-gnu.tar.xz &&\
    rm gcc-arm-8.3-2019.03-x86_64-aarch64-linux-gnu.tar.xz &&\
    wget -q -c https://developer.arm.com/-/media/Files/downloads/gnu-a/8.3-2019.03/binrel/gcc-arm-8.3-2019.03-x86_64-arm-eabi.tar.xz &&\
    tar xf gcc-arm-8.3-2019.03-x86_64-arm-eabi.tar.xz &&\
    rm gcc-arm-8.3-2019.03-x86_64-arm-eabi.tar.xz

ENV CC=/work/gcc-arm-8.3-2019.03-x86_64-aarch64-linux-gnu/bin/aarch64-linux-gnu-
ENV M0_CROSS_COMPILE=/work/gcc-arm-8.3-2019.03-x86_64-arm-eabi/bin/arm-eabi-

RUN wget -q https://github.com/ARM-software/arm-trusted-firmware/archive/refs/tags/v2.6.zip && \
    unzip -q v2.6.zip &&\
    rm v2.6.zip &&\
    cd arm-trusted-firmware-2.6/ && \
    make CROSS_COMPILE=${CC} realclean && \
    make CROSS_COMPILE=${CC} PLAT=sun50i_a64

# SPL compile
FROM basebuild as or1k

USER root

RUN ln -s /usr/lib/x86_64-linux-gnu/libmpfr.so.6 /usr/lib/x86_64-linux-gnu/libmpfr.so.4

WORKDIR /work

USER builder

RUN wget -q https://musl.cc/or1k-linux-musl-cross.tgz && \
    tar zxvf or1k-linux-musl-cross.tgz &&\
    rm or1k-linux-musl-cross.tgz

ENV PATH="${PATH}:/work/or1k-linux-musl-cross/bin"
ENV CROSS_COMPILE=or1k-linux-musl-

RUN wget -q https://github.com/crust-firmware/crust/archive/refs/tags/v0.5.zip &&\
    unzip -q v0.5.zip &&\
    rm v0.5.zip &&\
    cd crust-0.5 && \
    make orangepi_zero_plus_defconfig && \
    make scp 

# U-Boot compile 
FROM basebuild as uboot

# Set OCI-compliant labels
LABEL org.opencontainers.image.title="orange-pi-zero-plus-uboot" \
      org.opencontainers.image.version="2022.02" \
      org.opencontainers.image.description="A simple docker container to build u-boot image for OrangePI Zero Plus" \
      org.opencontainers.image.authors="Nurmukhamed Artykaly <nurmukhamed.artykaly@hdfilm.kz>" \
      org.opencontainers.image.licenses="GPLv3" \
      org.opencontainers.image.source="https://github.com/Nurmukhamed/OrangePIPC-Boot-From-SPIFLASH" \
      org.opencontainers.image.url="https://github.com/Nurmukhamed/OrangePIPC-Boot-From-SPIFLASH"

COPY --from=armtrustedfirmware /work/arm-trusted-firmware-2.6/build/sun50i_a64/release/bl31.bin /bl31.bin
COPY --from=or1k /work/crust-0.5/build/scp/scp.bin /scp.bin

# Disable Prompt During Packages Installation
ARG DEBIAN_FRONTEND=noninteractive

ENV ARCH=arm
ENV CROSS_COMPILE=aarch64-linux-gnu-
ENV BL31=/bl31.bin
ENV SCP=/scp.bin
ENV DEBIAN_FRONTEND=noninteractive

WORKDIR /work

USER builder

RUN wget -q https://github.com/u-boot/u-boot/archive/refs/tags/v2022.01.zip &&\
    unzip -q v2022.01.zip &&\
    rm v2022.01.zip &&\
    cd u-boot-2022.01 && \
    make distclean 

COPY config /work/u-boot-2022.01/.config 
COPY sun50i-h5-orangepi-zero-plus.dts /work/u-booti-2022.01/arch/arm/dts/sun50i-h5-orangepi-zero-plus.dts

RUN cd /work/u-boot-2022.01 &&\
    make

# Final image
FROM ubuntu:20.04 AS final
COPY --from=uboot /work/u-boot-2022.01/u-boot-sunxi-with-spl.bin /u-boot-sunxi-with-spl.bin

CMD ["/bin/sh", "-c", "cp /u-boot-sunxi-with-spl.bin /output"]

