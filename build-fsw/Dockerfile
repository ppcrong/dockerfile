FROM ubuntu:18.04

ENV IMAGE_NAME build-fsw:1.0.3314212643.0
ENV USER_PATH /root
ENV TOOL_PATH ${USER_PATH}/tool

# apt update and install linux-headers
ARG LINUX_HEADER_PACKAGE_ARG
ENV LINUX_HEADER_PACKAGE_DEFAULT linux-headers-5.4.0-144-generic
ENV LINUX_HEADER_PACKAGE ${LINUX_HEADER_PACKAGE_ARG:-${LINUX_HEADER_PACKAGE_DEFAULT}}
RUN apt update && apt install -y ${LINUX_HEADER_PACKAGE}

# apt install dependencies
RUN apt update && apt install -y \
    sudo \
    openssh-server \
    openssh-client \
    build-essential \
    gdb \
    aptitude \
    && rm -rf /var/lib/apt/lists/*

# Ubuntu 18.04 ships with cmake-3.10.2
# cfsext/tests/gtest needs cmake 3.14+
# test-asan needs cmake 3.16.3
RUN apt update && apt install -y \
    software-properties-common
RUN wget -O - https://apt.kitware.com/keys/kitware-archive-latest.asc 2>/dev/null | sudo apt-key add -
RUN apt-add-repository 'deb https://apt.kitware.com/ubuntu/ bionic main'
RUN apt update && apt install -y \
    cmake=3.16.3-0kitware1 cmake-data=3.16.3-0kitware1

# aptitude install dependencies
RUN aptitude update && aptitude install -y \
    llvm-8 \
    llvm-8-dev \
    clang-8 \
    libclang-8-dev

# create link for llvm8
RUN ln -s -f /usr/bin/llvm-config-8 /usr/bin/llvm-config

# show all llvm-config*
RUN ls -l /usr/bin/llvm-config*

# aptitude install dependencies (take long time)
RUN aptitude install -y \
    cmake \
    libblas-dev \
    liblapack-dev \
    gcc-multilib \
    g++-multilib \
    libgsl-dev \
    libboost-all-dev \
    libsocketcan-dev \
    openssh-server \
    mercurial \
    texinfo \
    libelf-dev

# aptitude install nasa trick dependencies (take long time)
RUN aptitude install -y \
    bison \
    clang \
    flex \
    git \
    llvm \
    make \
    maven \
    swig \
    cmake \
    curl \
    g++ \
    libx11-dev \
    libxml2-dev \
    libxt-dev \
    libmotif-common \
    libmotif-dev \
    python3-dev \
    zlib1g-dev \
    llvm-dev \
    libclang-dev \
    libudunits2-dev \
    libgtest-dev \
    openjdk-11-jdk \
    zip

# create tool folder
RUN mkdir -p ${TOOL_PATH}
WORKDIR ${TOOL_PATH}

####################################################################################
# install nasa trick
ENV PYTHON_VERSION 3
RUN git clone https://github.com/nasa/trick.git
WORKDIR ${TOOL_PATH}/trick
RUN git checkout 19.2.2
RUN ./configure

# make (take long time)
RUN make

# install
RUN make install

# back to tool folder
WORKDIR ${TOOL_PATH}

####################################################################################
# install libarmadillo
RUN wget https://sourceforge.net/projects/arma/files/armadillo-8.600.1.tar.xz
RUN tar -Jxf armadillo-8.600.1.tar.xz
WORKDIR ${TOOL_PATH}/armadillo-8.600.1
RUN cmake .
RUN make
RUN make install

# back to tool folder
WORKDIR ${TOOL_PATH}

####################################################################################
# install libethercat
RUN aptitude install -y autoconf
RUN hg clone http://hg.code.sf.net/p/etherlabmaster/code ecmaster
WORKDIR ${TOOL_PATH}/ecmaster
RUN hg update stable-1.5
RUN ./bootstrap
RUN ./configure --with-linux-dir=/usr/src/${LINUX_HEADER_PACKAGE} --disable-8139too --prefix=/usr
RUN make
RUN make install

# back to tool folder
WORKDIR ${TOOL_PATH}

####################################################################################
# install libpcan/libpcanfd
RUN aptitude install -y libpopt-dev can-utils
RUN wget -qO- https://www.peak-system.com/fileadmin/media/linux/files/peak-linux-driver-8.10.2.tar.gz | tar  xvz
WORKDIR ${TOOL_PATH}/peak-linux-driver-8.10.2

# apt install dependencies
RUN apt update && apt install -y libudev-dev udev

# clean
RUN make clean KERNEL_LOCATION=/usr/src/${LINUX_HEADER_PACKAGE}

# build
RUN make netdev KERNEL_LOCATION=/usr/src/${LINUX_HEADER_PACKAGE}

# install
RUN make -C driver install_files KERNEL_LOCATION=/usr/src/${LINUX_HEADER_PACKAGE}
RUN make -C lib install KERNEL_LOCATION=/usr/src/${LINUX_HEADER_PACKAGE}
RUN make -C test install KERNEL_LOCATION=/usr/src/${LINUX_HEADER_PACKAGE}
RUN make -C libpcanbasic install KERNEL_LOCATION=/usr/src/${LINUX_HEADER_PACKAGE}

# install (original)
# make install (error) consists of:
#   make -C driver install
#   make -C lib install
#   make -C test install
#   make -C libpcanbasic install

# back to user folder
WORKDIR ${USER_PATH}

####################################################################################
# install sdt dependencies
RUN apt install -y gawk \
    software-properties-common
RUN apt remove gcc-arm-none-eabi
RUN apt autoremove
RUN add-apt-repository ppa:team-gcc-arm-embedded/ppa
RUN apt update && apt install -y \
    gcc-arm-embedded

####################################################################################
# install SIL dependencies
RUN aptitude install -y gnuplot
RUN apt install -y locales locales-all
ENV LC_ALL en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8

####################################################################################
# install git-lfs
RUN apt install -y git-lfs

####################################################################################
# install git-repo
RUN mkdir -p ~/.bin
RUN curl https://storage.googleapis.com/git-repo-downloads/repo > ~/.bin/repo
RUN chmod a+rx ~/.bin/repo
RUN sudo echo "PATH=\"\${HOME}/.bin:\${PATH}\"" | sudo tee -a ~/.bashrc

####################################################################################
# install system dependencies
RUN apt install -y cpio \
    rsync \
    bc

####################################################################################
# install test-asan dependencies
# iproute2 for cmd "ip a" to check ip address
# ifupdown for test-asan target using /etc/init.d/networking
RUN apt update && apt install -y \
    iproute2 \
    ifupdown

####################################################################################
# remove all tool source
RUN rm -rf ${TOOL_PATH}

CMD ["bash"]
