FROM buildpack-deps:bionic-curl

# Do not exclude man pages & other documentation
# Reinstall all currently installed packages in order to get the man pages back
RUN rm /etc/dpkg/dpkg.cfg.d/excludes && apt-get update && \
  dpkg -l | grep ^ii | cut -d' ' -f3 | xargs apt-get install -y --reinstall && \
  rm -r /var/lib/apt/lists/*

RUN set -ex; \
  apt-get update; \
  DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
  ######## Docker community block begin ########
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
  ######## Docker community block end ########
  ######## Begin Tools in Ubuntu Standard ########
  gawk \
  debianutils \
  bash \
  bsdutils \
  coreutils \
  whois \
  dash \
  tar \
  diffutils \
  findutils \
  grep \
  gzip \
  hostname \
  libpam-runtime \
  debconf \
  libpam-modules-bin \
  ncurses-bin \
  sed \
  less \
  apt \
  man-db \
  perl \
  sensible-utils \
  mime-support \
  readline-common \
  groff-base \
  lsof \
  autotools-dev \
  gnupg2 \
  make-doc \
  lsb-base \
  time \
  info \
  install-info \
  telnet \
  ftp \
  whiptail \
  bash-completion \
  dnsutils \
  command-not-found \
  cpio \
  ed \
  gettext \
  iproute2 \
  irqbalance \
  kmod \
  lshw \
  mtr-tiny \
  netcat-openbsd \
  psmisc \
  rsync \
  strace \
  ######## End Tools in Ubuntu Standard ########
  ####### From Netlify #######
  advancecomp \
  expect \
  git-lfs \
  jq \
  graphviz \
  jpegoptim \
  optipng \
  pngcrush \
  python3.8 \
  python3.8-dev \
  ssh \
  # PHP Install
  php7.2 \
  php7.2-xml \
  php7.2-mbstring \
  php7.2-gd \
  php7.2-sqlite3 \
  php7.2-curl \
  php7.2-zip \
  software-properties-common \
  sqlite3 \
  xvfb \
  # END PHP
  virtualenv \
  tree \
  zip \
  ### End Netlify ###
  && DEBIAN_FRONTEND=noninteractive apt-get install -y \
  git \
  openssh-client \
  procps \
  vim \
  neovim \
  wget \
  sudo \
  htop \
  man \
  net-tools \
  locales \
  #### Clean Up apt-cache ####
  && apt-get autoremove --purge -y && apt-get -y clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
  apt-get autoremove -y && \
  unset DEBIAN_FRONTEND

RUN locale-gen en_US.UTF-8
# We unfortunately cannot use update-locale because docker will not use the env variables
# configured in /etc/default/locale so we need to set it manually.
ENV LC_ALL=en_US.UTF-8 \
  LANGUAGE=en_US:en \
  LANG=en_US.UTF-8
