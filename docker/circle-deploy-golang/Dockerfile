# Image to have all tools to test golang code in circleci
# Includes: gcloud sdk, go, gcloud emulators
FROM google/cloud-sdk:alpine

RUN apk add --no-cache git go docker
RUN gcloud -q components install beta && gcloud -q components update

# Configure Go
ENV GOROOT="/usr/lib/go" \
    GOPATH="/go" \
    PATH="/go/bin:$PATH"

# Install kubectl
RUN curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.16.0/bin/linux/amd64/kubectl
RUN echo "4fc8a7024ef17b907820890f11ba7e59a6a578fa91ea593ce8e58b3260f7fb88  kubectl" | sha256sum -c -
RUN mv kubectl /usr/local/bin
RUN chmod +x /usr/local/bin/kubectl

# Install JQ
RUN curl -LO https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64
RUN mv jq-linux64 jq
RUN echo "af986793a515d500ab2d35f8d2aecd656e764504b789b66d7e1a0b727a124c44  jq" | sha256sum -c -
RUN mv jq /usr/local/bin
RUN chmod +x /usr/local/bin/jq
