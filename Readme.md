## Docker images

This repository contains the Dockerfiles for building the GPU images for detectcnn

The docker images contain:
- a running `dede` server ready to be used, no install required
- `googlenet` and `resnet_50` pre-trained image classification models, in `/opt/models/`

This allows to run the container and set an image classification model based on deep (residual) nets in two short command line calls.

#### Building the image

Example goes with the image:
```
docker build -t detectcnn --no-cache .
```

#### Running the GPU image

This requires [nvidia-docker](https://github.com/NVIDIA/nvidia-docker) in order for the local GPUs to be made accessible by the container.

The following steps are required:

- install `nvidia-docker`: https://github.com/NVIDIA/nvidia-docker
- run with
```
nvidia-docker run -d -p 8080:8080 detectcnn
```

Notes:
- `nvidia-docker` requires docker >= 1.9

To test on image classification on GPU:
```
curl -X PUT "http://localhost:8080/services/imageserv" -d "{\"mllib\":\"caffe\",\"description\":\"image classification service\",\"type\":\"supervised\",\"parameters\":{\"input\":{\"connector\":\"image\"},\"mllib\":{\"nclasses\":1000}},\"model\":{\"repository\":\"/opt/models/ggnet/\"}}"
{"status":{"code":201,"msg":"Created"}}
```
and
```
curl -X POST "http://localhost:8080/predict" -d "{\"service\":\"imageserv\",\"parameters\":{\"input\":{\"width\":224,\"height\":224},\"output\":{\"best\":3},\"mllib\":{\"gpu\":true}},\"data\":[\"http://i.ytimg.com/vi/0vxOhd4qlnA/maxresdefault.jpg\"]}"
```

Try the `POST` call twice: first time loads the net so it takes slightly below a second, then second call should yield a `time` around 100ms as reported in the output JSON.

#### Access to server logs

To look at server logs, use 
```
docker logs -f <container name>
```
where <container name> can be obtained via `docker ps`

- look for container:
```
> docker ps
CONTAINER ID        IMAGE                  COMMAND                  CREATED              STATUS              PORTS                    NAMES
d9944734d5d6        detectcnn   "/bin/sh -c './dede -"   17 seconds ago       Up 16 seconds       0.0.0.0:8080->8080/tcp   loving_shaw
```

- access server logs:
```
> docker logs -f loving_shaw 

DeepDetect [ commit 4e2c9f4cbd55eeba3a93fae71d9d62377e91ffa5 ]
Running DeepDetect HTTP server on 0.0.0.0:8080
```
