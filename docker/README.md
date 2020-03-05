# quibi-public Docker Images

This directory contains `Dockerfile`s for docker images hosted on our public registry `gcr.io/quibi-public`. 
Below are descriptions for each as well as any additional instructions to be able to build and publish new versions.

## Build Enhancements for Docker
Some images require authentication to private Quibi git repositories. In order to facilitate this, we use SSH forwarding 
from your local ssh agent.

To enable buildkit builds add the following environment variable to your development environment:
```
DOCKER_BUILDKIT=1
```
Alternatively, set it every time you run docker build:
```
> DOCKER_BUILDKIT=1 docker build .
``` 
[circle-schema](circle-schema/Dockerfile) uses this functionality. See its `Dockerfile` for reference. Of note are the lines:
```
# syntax=docker/dockerfile:1.0.2-experimental
... 
RUN --mount=type=ssh,mode=777 GO111MODULE=on go get github.com/quibitv/twirp/protoc-gen-twirp@30d1365 # quibi branch
```

Once the `Dockerfile` is created. The image can be created by using the `--ssh` option for connectivity with the SSH agent:
```
docker build --ssh default -t gcr.io/quibi-public/circle-schema:5 .
```

For more details see [Build Enhancements for Docker](https://docs.docker.com/develop/develop-images/build_enhancements/)

## Docker Registry
For current tags visit
https://console.cloud.google.com/gcr/images/quibi-public?project=quibi-public&organizationId=752457394362

## Images
### circle-schema
Docker image to use on CircleCI to test proto schema projects. Contains the following tools:
- golang
- prototool (protobuf is bundled with this tool)
- protoc-gen-go 
- ptotoc-gen-twirp (quibi fork)

Build:
```
> cd circle-schema
> docker build --ssh default -t gcr.io/quibi-public/circle-schema:${new_version} .
> docker push gcr.io/quibi-public/circle-schema:${new_version}
```

### circle-test-golang
Image to have all tools to test golang code in CircleCI. Contains the following tools:
- golang
- google-cloud-sdk
- google cloud components:
  - beta
  - pubsub-emulator
  - cloud-datastore-emulator

Build:
```
> cd circle-test-golang
> docker build -t gcr.io/quibi-public/circle-test-golang:${new_version} .
> docker push gcr.io/quibi-public/circle-test-golang:${new_version}
```

### circle-deploy-golang
Image with all tools needed to deploy services to Google Cloud. Containes:
- golang
- google-cloud-sdk
- google cloud components:
  - kubectl

Build:
```
> cd circle-deploy-golang
> docker build -t gcr.io/quibi-public/circle-deploy-golang:${new_version} .
> docker push gcr.io/quibi-public/circle-deploy-golang:${new_version}
```
