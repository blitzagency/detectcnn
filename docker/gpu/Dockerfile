FROM nvidia/cuda:7.5-cudnn4-devel

RUN echo 'Building GPU DetectCNN image'

MAINTAINER Matt Zeiger "mzeiger@blitzagency.com"

LABEL description="Deep learning server & API / GPU version"

RUN ln -sf /dev/stdout /var/log/detectcnn.log
RUN ln -sf /dev/stderr /var/log/detectcnn.log

RUN useradd -ms /bin/bash dd

RUN apt-get update && apt-get install -y git cmake build-essential libgoogle-glog-dev libgflags-dev libeigen3-dev libopencv-dev libcppnetlib-dev libboost-dev libcurlpp-dev libcurl4-openssl-dev protobuf-compiler libopenblas-dev libhdf5-dev libprotobuf-dev libleveldb-dev libsnappy-dev liblmdb-dev libutfcpp-dev wget

WORKDIR /opt
RUN git clone https://github.com/blitzagency/detectcnn.git && cd detectcnn && mkdir build

WORKDIR /opt/detectcnn/build
RUN cmake .. -DUSE_CUDNN=ON -DUSE_XGBOOST=ON
RUN make

# external volume to be mapped, e.g. for models or training data
RUN mkdir /data
VOLUME ["/data"]

# include a few image models within the image
RUN mkdir /opt/models
WORKDIR /opt/models
RUN mkdir ggnet && cd ggnet && wget http://www.deepdetect.com/models/ggnet/bvlc_googlenet.caffemodel
RUN mkdir resnet_50 && cd resnet_50 && wget http://www.deepdetect.com/models/resnet/ResNet-50-model.caffemodel && wget http://www.deepdetect.com/models/resnet/ResNet_mean.binaryproto
RUN cp /opt/detectcnn/datasets/imagenet/corresp_ilsvrc12.txt /opt/models/ggnet/corresp.txt
RUN cp /opt/detectcnn/datasets/imagenet/corresp_ilsvrc12.txt /opt/models/resnet_50/corresp.txt
RUN cp /opt/detectcnn/templates/caffe/googlenet/*prototxt /opt/models/ggnet/
RUN cp /opt/detectcnn/templates/caffe/resnet_50/*prototxt /opt/models/resnet_50/

# permissions
RUN chown -R dd:dd /opt/detectcnn
RUN chown -R dd:dd /opt/models

WORKDIR /opt/detectcnn/build/main
CMD ./dede -host 0.0.0.0
EXPOSE 8080