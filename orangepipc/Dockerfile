# Download base image ubuntu 20.04
FROM ubuntu:20.04

# Set OCI-compliant labels
LABEL org.opencontainers.image.title="orange-pi-pc-uboot" \
      org.opencontainers.image.version="2022.02" \
      org.opencontainers.image.description="A simple docker container to build u-boot image for OrangePI PC" \
      org.opencontainers.image.authors="Nurmukhamed Artykaly <nurmukhamed.artykaly@hdfilm.kz>" \
      org.opencontainers.image.licenses="GPLv3" \
      org.opencontainers.image.source="https://github.com/Nurmukhamed/OrangePIPC-Boot-From-SPIFLASH" \
      org.opencontainers.image.url="https://github.com/Nurmukhamed/OrangePIPC-Boot-From-SPIFLASH"

# Disable Prompt During Packages Installation
ARG DEBIAN_FRONTEND=noninteractive
ARG GROUP=builder
ARG USER=builder

# Update Ubuntu Software repository
RUN apt update -qq -y

# Install packages
RUN apt install -qq -y \
      build-essential \
      gcc-arm-linux-gnueabihf \
      bc \
      bison \
      flex \
      swig \
      git \
      python3 \
      python3-distutils \
      python3-dev \
      libssl-dev \
      python3-setuptools \
      wget \ 
      zip unzip &&\
    rm -rf /var/lib/apt/lists/* &&\
    apt clean

# Create group and user
RUN groupadd ${GROUP} &&\
    useradd -ms /bin/bash -g ${GROUP} ${USER} &&\
    mkdir /app &&\
    mkdir /output &&\
    chown ${USER}:${GROUP} /app /output &&\
    chmod 0755 /app &&\
    chmod 0777 /output

USER ${USER}

# Everything is happen in /app
WORKDIR /app

# Download git repository
RUN wget -q https://github.com/u-boot/u-boot/archive/refs/tags/v2022.01.zip &&\
    unzip -q  v2022.01.zip &&\
    cd u-boot-2022.01 && \
    make distclean && \
    make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- orangepi_pc_defconfig

# Copying dts file to source
COPY sun8i-h3-orangepi-pc.dts u-boot-2022.01/arch/arm/dts/sun8i-h3-orangepi-pc.dts
COPY config u-boot-2022.01/.config

# Make image
RUN cd u-boot-2022.01 &&\
    make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- && \
    cp ./u-boot-sunxi-with-spl.bin /output

CMD ["/bin/sh", "-c", "cd /app/u-boot-2022.01 && make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- && cp ./u-boot-sunxi-with-spl.bin /output"]
