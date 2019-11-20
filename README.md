# docker-polyglot

A docker image for multilingual developers.

## Why

docker-polyglot provides a base image for development of multi-language projects. When a microservice project is containerized, it is standard for each language runtime to exist in its own container process. However, when building this software, it is more convenient for all languages to be available in one environment. This allows for things like IDE language servers and other development tools to work seemlessly. It also simplifies development because a single environment is used for bootstrapping all projects.

## How

Consider this container your development virtual machine. Download and use it as a base image to customize the environment.
