FROM ubuntu:18.04

ENV IMAGE_NAME temp:1.0.0.0
ENV USER_PATH /root
ENV TOOL_PATH ${USER_PATH}/tool
WORKDIR ${TOOL_PATH}

# apt update and install linux-headers
RUN apt update && apt install -y \
    wget

# install libpcan/libpcanfd
RUN wget -qO- https://www.peak-system.com/fileadmin/media/linux/files/peak-linux-driver-8.10.2.tar.gz | tar  xvz
WORKDIR ${TOOL_PATH}/peak-linux-driver-8.10.2

####################################################################################
# install git-lfs
RUN apt install -y git-lfs

WORKDIR ${USER_PATH}

####################################################################################
# remove all tool source
RUN rm -rf ${TOOL_PATH}

CMD ["bash"]
