# Image to have all tools to test golang code in circleci
# Includes: gcloud sdk, go, gcloud emulators
FROM google/cloud-sdk:270.0.0-alpine

RUN apk --update add openjdk8-jre
RUN apk add --no-cache git \
    make \
    musl-dev
RUN apk add --no-cache -X http://dl-cdn.alpinelinux.org/alpine/edge/community go=1.13.3-r1
RUN gcloud -q components install beta pubsub-emulator cloud-datastore-emulator
RUN gcloud -q components update

# Configure Go
ENV GOROOT="/usr/lib/go" \
    GOPATH="/go" \
    PATH="/go/bin:$PATH"

RUN mkdir -p ${GOPATH}/src ${GOPATH}/bin
RUN curl -sfL https://install.goreleaser.com/github.com/golangci/golangci-lint.sh | sh -s -- -b $(go env GOPATH)/bin v1.16.0

# Install JQ
RUN curl -LO https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64
RUN mv jq-linux64 jq
RUN echo "af986793a515d500ab2d35f8d2aecd656e764504b789b66d7e1a0b727a124c44  jq" | sha256sum -c -
RUN mv jq /usr/local/bin
RUN chmod +x /usr/local/bin/jq