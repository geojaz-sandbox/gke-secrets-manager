# syntax=docker/dockerfile:1.0.2-experimental
# NOTE: Ensure your ssh agent has your key registered.  check with `ssh-add -L`  if not, add it with `ssh-add -K [keyfile]`

FROM swift:5.0.1-xenial as swift
RUN apt-get update && apt-get -y install curl
RUN curl -L https://github.com/seanliu1/swift-protobuf/archive/addExcludeNestedProperties.tar.gz | tar -xz
RUN make -C swift-protobuf-addExcludeNestedProperties

FROM circleci/golang:1.12.4-stretch

USER root

COPY --from=swift swift-protobuf-addExcludeNestedProperties/.build/x86_64-unknown-linux/debug/protoc-gen-swift /usr/bin
COPY --from=swift /usr/lib/swift/linux /usr/lib/swift/linux
RUN echo /usr/lib/swift/linux >> /etc/ld.so.conf

# Installing lib dependencies (tree, java, gradle)
RUN apt-get update && apt-get install -y tree openjdk-8-jdk libxml2 python-setuptools

RUN wget https://services.gradle.org/distributions/gradle-5.4.1-bin.zip -P /tmp \
    && unzip -d /opt/gradle /tmp/gradle-*.zip

ENV PATH="/opt/gradle/gradle-5.4.1/bin:${PATH}"

# Setup ssh
RUN mkdir -p -m 0700 ~/.ssh && ssh-keyscan github.com >> ~/.ssh/known_hosts \
    && git config --global url."git@github.com:".insteadOf "https://github.com/"

# Setup Proto dependencies
RUN curl -sSL https://github.com/uber/prototool/releases/download/v1.6.0/prototool-$(uname -s)-$(uname -m) -o /go/bin/prototool
RUN chmod +x /go/bin/prototool
RUN prototool cache update

ENV GO111MODULE on
RUN --mount=type=ssh,mode=777 go get github.com/golang/protobuf/protoc-gen-go@v1.2.0 \
    && go get github.com/quibitv/twirp-gen/protoc-gen-twirp@96fe9d1 \
    && go get github.com/quibitv/twirp-gen/protoc-gen-twirp_java_jaxrs@96fe9d1 \
    && go get github.com/quibitv/twirp-gen/protoc-gen-twirp_python@96fe9d1 \
    && go get github.com/quibitv/twirp-gen/protoc-gen-twirp_swift@96fe9d1 \
    && rm -rf /go/pkg

# Installing JFrog CLI for publishing
RUN curl -fL https://getcli.jfrog.io | sh && sudo mv ./jfrog /usr/local/bin/ && chmod 0555 /usr/local/bin/jfrog

# Install node/npm for JS prototool integration.
RUN curl -sL https://deb.nodesource.com/setup_12.x | bash -
RUN apt-get install -y nodejs

USER circleci
