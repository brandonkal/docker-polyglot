FROM buildpack-deps:bionic-curl

RUN apt-get update && apt-get install -y --no-install-recommends \
  git \
  openssh-client \
  procps \
  vim \
  neovim \
  curl \
  wget \
  lsof \
  inetutils-ping \
  sudo \
  htop \
  man \
  ripgrep \
  net-tools \
  locales; \
  rm -rf /var/lib/apt/lists/*

RUN locale-gen en_US.UTF-8
# We unfortunately cannot use update-locale because docker will not use the env variables
# configured in /etc/default/locale so we need to set it manually.
ENV LC_ALL=en_US.UTF-8