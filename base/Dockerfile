FROM buildpack-deps:bionic-curl

RUN set -ex; \
  apt-get update; \
  apt-get install -y --no-install-recommends \
  autoconf \
  automake \
  bzip2 \
  dpkg-dev \
  file \
  g++ \
  gcc \
  imagemagick \
  libbz2-dev \
  libc6-dev \
  libcurl4-openssl-dev \
  libdb-dev \
  libevent-dev \
  libffi-dev \
  libgdbm-dev \
  libglib2.0-dev \
  libgmp-dev \
  libjpeg-dev \
  libkrb5-dev \
  liblzma-dev \
  libmagickcore-dev \
  libmagickwand-dev \
  libmaxminddb-dev \
  libncurses5-dev \
  libncursesw5-dev \
  libpng-dev \
  libpq-dev \
  libreadline-dev \
  libsqlite3-dev \
  libssl-dev \
  libtool \
  libwebp-dev \
  libxml2-dev \
  libxslt-dev \
  libyaml-dev \
  make \
  patch \
  unzip \
  xz-utils \
  zlib1g-dev \
  \
  # https://lists.debian.org/debian-devel-announce/2016/09/msg00000.html
  $( \
  # if we use just "apt-cache show" here, it returns zero because "Can't select versions from package 'libmysqlclient-dev' as it is purely virtual", hence the pipe to grep
  if apt-cache show 'default-libmysqlclient-dev' 2>/dev/null | grep -q '^Version:'; then \
  echo 'default-libmysqlclient-dev'; \
  else \
  echo 'libmysqlclient-dev'; \
  fi \
  ) \
  ; \
  rm -rf /var/lib/apt/lists/*

RUN apt-get update && apt-get install -y \
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
  net-tools \
  locales && \
  rm -rf /var/lib/apt/lists/*

RUN locale-gen en_US.UTF-8
# We unfortunately cannot use update-locale because docker will not use the env variables
# configured in /etc/default/locale so we need to set it manually.
ENV LC_ALL=en_US.UTF-8 \
  LANGUAGE=en_US:en \
  LANG=en_US.UTF-8
