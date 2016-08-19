FROM ubuntu:14.04

RUN echo 'building CPU DetectCNN image'

MAINTAINER Matt Zeiger "mzeiger@blitzagency.com"
LABEL description="DetectCNN deep learning server & API / CPU version"

RUN ln -sf /dev/stdout /var/log/detectcnn.log
RUN ln -sf /dev/stderr /var/log/detectcnn.log

RUN useradd -ms /bin/bash dd

RUN apt-get update && apt-get install -y git cmake build-essential libgoogle-glog-dev libgflags-dev libeigen3-dev libopencv-dev libcppnetlib-dev libboost-dev libcurlpp-dev libcurl4-openssl-dev protobuf-compiler libopenblas-dev libhdf5-dev libprotobuf-dev libleveldb-dev libsnappy-dev liblmdb-dev libutfcpp-dev wget

WORKDIR /opt
RUN git clone https://github.com/blitzagency/detectcnn.git && cd detectcnn && mkdir build

WORKDIR /opt/detectcnn/build
RUN cmake .. -DUSE_XGBOOST=ON
RUN make

# external volume to be mapped, e.g. for models or training data
RUN mkdir /data
VOLUME ["/data"]

# include an image model within the image
RUN mkdir /opt/models
WORKDIR /opt/models
RUN mv /opt/detectcnn/models/ggnet /opt/models

# permissions
RUN chown -R dd:dd /opt/detectcnn
RUN chown -R dd:dd /opt/models

USER dd
WORKDIR /opt/detectcnn/build/main
CMD ./dede -host 0.0.0.0
EXPOSE 8080