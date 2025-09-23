FROM mcr.microsoft.com/devcontainers/base:ubuntu-24.04

RUN apt-get update

# install docker from an official shellscript installer
RUN curl -fsSL https://get.docker.com -o install-docker.sh
RUN sh install-docker.sh

RUN apt-get install iputils-ping net-tools