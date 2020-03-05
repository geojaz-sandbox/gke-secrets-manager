# example-schema

This repo is a starting point for new repositories to draw on the current "best practices" for CI, project setup, and review of shared protobuf schema for use between services. Additionally, if anyone wants to propose a platform-wide process changes we can discuss them here and then disseminate to derived projects. 

Currently there are three common distribution options. Individual projects can choose which they prefer:

1) Artifact Distribution
  * CI builds and publishes artifacts from a project-schema repo and all interested projects import the artifact
  * *example: https://github.com/quibitv/payments-schema*
2) Submodule
  * Interested projects use submodule to reference the project-schema repo and are responsible for compiling
  * *example: https://github.com/quibitv/qlient-api-schema*
3) Manual
  * Interested projects manually copy compiled artifacts into their respective source trees
  * *example: https://github.com/quibitv/cms/tree/master/api*

## Continuous Integration Docker Images
To facilitate the build and test processes on CircleCI, the [docker](docker) repository contains images hosted on our public registry. See the
[README.md](docker/README.md) there for more details.