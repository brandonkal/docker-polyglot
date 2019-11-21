FROM brandonkal/docker-polyglot:base

### Default user so we don't run everything as root
RUN adduser --gecos '' --uid 1000 --disabled-password user && \
  echo "user ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/nopasswd

#  88888888ba   88        88  88888888ba
#  88      "8b  88        88  88      "8b
#  88      ,8P  88        88  88      ,8P
#  88aaaaaa8P'  88aaaaaaaa88  88aaaaaa8P'
#  88""""""'    88""""""""88  88""""""'
#  88           88        88  88
#  88           88        88  88
#  88           88        88  88

# Installed in brandonkal/polyglot:base
RUN wget -nv https://raw.githubusercontent.com/composer/getcomposer.org/6e5e46fc7cab0015d5961f54bb39be1f4b330093/web/installer -O - | php -- --quiet && \
  mv composer.phar /usr/local/bin/composer

#  88888888ba                88
#  88      "8b               88
#  88      ,8P               88
#  88aaaaaa8P'  88       88  88,dPPYba,   8b       d8
#  88""""88'    88       88  88P'    "8a  `8b     d8'
#  88    `8b    88       88  88       d8   `8b   d8'
#  88     `8b   "8a,   ,a88  88b,   ,a8"    `8b,d8'
#  88      `8b   `"Ruby'Y8  8Y"Ybbd8"'       Y88'
#                                             d8'
# Taken From:                                d8'
# https://github.com/docker-library/ruby/blob/master/2.6/stretch/Dockerfile
# https://github.com/docker-library/ruby/blob/4ba4a2d115ac496524d1cca392dbe9560b7b0134/2.6/buster/Dockerfile
# Modifications:
# Changed FROM to be compatible with `buildfrom.sh`
# Removed CMD ["irb"] at the end.

# skip installing gem documentation
RUN set -eux; \
  mkdir -p /usr/local/etc; \
  { \
  echo 'install: --no-document'; \
  echo 'update: --no-document'; \
  } >> /usr/local/etc/gemrc

ENV RUBY_MAJOR 2.6
ENV RUBY_VERSION 2.6.5
ENV RUBY_DOWNLOAD_SHA256 d5d6da717fd48524596f9b78ac5a2eeb9691753da5c06923a6c31190abe01a62

# some of ruby's build scripts are written in ruby
#   we purge system ruby later to make sure our final image uses what we just built
RUN set -eux; \
  \
  savedAptMark="$(apt-mark showmanual)"; \
  apt-get update; \
  apt-get install -y --no-install-recommends \
  bison \
  dpkg-dev \
  libgdbm-dev \
  ruby \
  ; \
  rm -rf /var/lib/apt/lists/*; \
  \
  wget -O ruby.tar.xz "https://cache.ruby-lang.org/pub/ruby/${RUBY_MAJOR%-rc}/ruby-$RUBY_VERSION.tar.xz"; \
  echo "$RUBY_DOWNLOAD_SHA256 *ruby.tar.xz" | sha256sum --check --strict; \
  \
  mkdir -p /usr/src/ruby; \
  tar -xJf ruby.tar.xz -C /usr/src/ruby --strip-components=1; \
  rm ruby.tar.xz; \
  \
  cd /usr/src/ruby; \
  \
  # hack in "ENABLE_PATH_CHECK" disabling to suppress:
  #   warning: Insecure world writable dir
  { \
  echo '#define ENABLE_PATH_CHECK 0'; \
  echo; \
  cat file.c; \
  } > file.c.new; \
  mv file.c.new file.c; \
  \
  autoconf; \
  gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)"; \
  ./configure \
  --build="$gnuArch" \
  --disable-install-doc \
  --enable-shared \
  ; \
  make -j "$(nproc)"; \
  make install; \
  \
  apt-mark auto '.*' > /dev/null; \
  apt-mark manual $savedAptMark > /dev/null; \
  find /usr/local -type f -executable -not \( -name '*tkinter*' \) -exec ldd '{}' ';' \
  | awk '/=>/ { print $(NF-1) }' \
  | sort -u \
  | xargs -r dpkg-query --search \
  | cut -d: -f1 \
  | sort -u \
  | xargs -r apt-mark manual \
  ; \
  apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
  \
  cd /; \
  rm -r /usr/src/ruby; \
  # verify we have no "ruby" packages installed
  ! dpkg -l | grep -i ruby; \
  [ "$(command -v ruby)" = '/usr/local/bin/ruby' ]; \
  # rough smoke test
  ruby --version; \
  gem --version; \
  bundle --version

# install things globally, for great justice
# and don't create ".bundle" in all our apps
ENV GEM_HOME /usr/local/bundle
ENV BUNDLE_PATH="$GEM_HOME" \
  BUNDLE_SILENCE_ROOT_WARNING=1 \
  BUNDLE_APP_CONFIG="$GEM_HOME"
# path recommendation: https://github.com/bundler/bundler/pull/6469#issuecomment-383235438
ENV PATH $GEM_HOME/bin:$BUNDLE_PATH/gems/bin:$PATH
# adjust permissions of a few directories for running "gem install" as an arbitrary user
RUN mkdir -p "$GEM_HOME" && chmod 777 "$GEM_HOME"
# (BUNDLE_PATH = GEM_HOME, no need to mkdir/chown both)
#
#  88888888ba                         88
#  88      "8b                 ,d     88
#  88      ,8P                 88     88
#  88aaaaaa8P'  8b       d8  MM88MMM  88,dPPYba,    ,adPPYba,   8b,dPPYba,
#  88""""""'    `8b     d8'    88     88P'    "8a  a8"     "8a  88P'   `"8a
#  88            `8b   d8'     88     88       88  8b       d8  88       88
#  88             `8b,d8'      88,    88       88  "8a,   ,a8"  88       88
#  88               Y88'       "Y888  88       88   `"YbbdP"'   88       88
#                   d8'

# Install a default pip. Note that for projects using a virtualenv with pipenv is cleaner.
# if this is called "PIP_VERSION", pip explodes with "ValueError: invalid truth value '<VERSION>'"
ENV PYTHON_PIP_VERSION 19.3.1
# https://github.com/pypa/get-pip
ENV PYTHON_GET_PIP_URL https://github.com/pypa/get-pip/raw/ffe826207a010164265d9cc807978e3604d18ca0/get-pip.py
ENV PYTHON_GET_PIP_SHA256 b86f36cc4345ae87bfd4f10ef6b2dbfa7a872fbff70608a1e43944d283fd0eee

RUN set -ex; \
  \
  wget -O get-pip.py "$PYTHON_GET_PIP_URL"; \
  echo "$PYTHON_GET_PIP_SHA256 *get-pip.py" | sha256sum --check --strict -; \
  \
  python get-pip.py \
  --disable-pip-version-check \
  --no-cache-dir \
  "pip==$PYTHON_PIP_VERSION" \
  ; \
  pip --version; \
  \
  find /usr/local -depth \
  \( \
  \( -type d -a \( -name test -o -name tests -o -name idle_test \) \) \
  -o \
  \( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
  \) -exec rm -rf '{}' +; \
  rm -f get-pip.py

#  888b      88                        88                      88   ad88888ba
#  8888b     88                        88                      88  d8"     "8b
#  88 `8b    88                        88                      88  Y8,
#  88  `8b   88   ,adPPYba,    ,adPPYb,88   ,adPPYba,          88  `Y8aaaaa,
#  88   `8b  88  a8"     "8a  a8"    `Y88  a8P_____88          88    `"""""8b,
#  88    `8b 88  8b       d8  8b       88  8PP"""""""          88          `8b
#  88     `8888  "8a,   ,a8"  "8a,   ,d88  "8b,   ,aa  88,   ,d88  Y8a     a8P
#  88      `888   `"YbbdP"'    `"8bbdP"Y8   `"Ybbd8"'   "Y8888P"    "Y88888P"
#

RUN curl -o- -L https://yarnpkg.com/install.sh > /usr/local/bin/yarn-installer.sh

# Install node.js
USER user
RUN git clone https://github.com/creationix/nvm.git ~/.nvm && \
  cd ~/.nvm && \
  git checkout v0.34.0 && \
  cd /

ENV ELM_VERSION=0.19.0-bugfix6
ENV YARN_VERSION=1.13.0

RUN /bin/bash -c ". ~/.nvm/nvm.sh && \
  nvm install 12 && nvm use 12 && \
  bash /usr/local/bin/yarn-installer.sh --version $YARN_VERSION && \
  nvm alias default node && nvm cache clear"

USER root

#
#         db                                 88  88           88
#        d88b                                ""  88           88
#       d8'`8b                                   88           88
#      d8'  `8b      8b,dPPYba,   ,adPPYba,  88  88,dPPYba,   88   ,adPPYba,
#     d8YaaaaY8b     88P'   `"8a  I8[    ""  88  88P'    "8a  88  a8P_____88
#    d8""""""""8b    88       88   `"Y8ba,   88  88       d8  88  8PP"""""""
#   d8'        `8b   88       88  aa    ]8I  88  88b,   ,a8"  88  "8b,   ,aa
#  d8'          `8b  88       88  `"YbbdP"'  88  8Y"Ybbd8"'   88   `"Ybbd8"'
#
RUN pip install ansible ansible-lint pipenv
#
#    ,ad8888ba,
#   d8"'    `"8b
#  d8'
#  88              ,adPPYba,
#  88      88888  a8"     "8a
#  Y8,        88  8b       d8
#   Y8a.    .a88  "8a,   ,a8"
#    `"Y88888P"    `"YbbdP"'
#
# Taken From https://raw.githubusercontent.com/docker-library/golang/master/1.13/buster/Dockerfile
# https://github.com/docker-library/golang/blob/4cd30a13eca195db17474df090d84d2901ddf3d6/1.13/buster/Dockerfile
# Modifications:
# Changed FROM to be compatible with `buildfrom.sh`.
# Removed WORKDIR.
# Removed GOPATH.

# gcc for cgo
RUN apt-get update && apt-get install -y --no-install-recommends \
  g++ \
  gcc \
  libc6-dev \
  make \
  pkg-config \
  && rm -rf /var/lib/apt/lists/*

ENV GOLANG_VERSION 1.13.4

RUN set -eux; \
  \
  # this "case" statement is generated via "update.sh"
  dpkgArch="$(dpkg --print-architecture)"; \
  case "${dpkgArch##*-}" in \
  amd64) goRelArch='linux-amd64'; goRelSha256='692d17071736f74be04a72a06dab9cac1cd759377bd85316e52b2227604c004c' ;; \
  armhf) goRelArch='linux-armv6l'; goRelSha256='9f76e6353c9ae2dcad1731b7239531eb8be2fe171f29f2a9c5040945a930fd41' ;; \
  arm64) goRelArch='linux-arm64'; goRelSha256='8b8d99eb07206f082468fb4d0ec962a819ae45d54065fc1ed6e2c502e774aaf0' ;; \
  i386) goRelArch='linux-386'; goRelSha256='497934398ca57c7c207ce3388f099823923b4c7b74394d6ed64cd2d3751aecb8' ;; \
  ppc64el) goRelArch='linux-ppc64le'; goRelSha256='815bf3c7100e73cfac50c4a07c8eeb4b0458a49ffa0e13a74a6cf7ad8e2a6499' ;; \
  s390x) goRelArch='linux-s390x'; goRelSha256='efc6947e8eb0a6409f4c8ba62b00ae4e54404064bc221df1b73364a95945a350' ;; \
  *) goRelArch='src'; goRelSha256='95dbeab442ee2746b9acf0934c8e2fc26414a0565c008631b04addb8c02e7624'; \
  echo >&2; echo >&2 "warning: current architecture ($dpkgArch) does not have a corresponding Go binary release; will be building from source"; echo >&2 ;; \
  esac; \
  \
  url="https://golang.org/dl/go${GOLANG_VERSION}.${goRelArch}.tar.gz"; \
  wget -O go.tgz "$url"; \
  echo "${goRelSha256} *go.tgz" | sha256sum -c -; \
  tar -C /usr/local -xzf go.tgz; \
  rm go.tgz; \
  \
  if [ "$goRelArch" = 'src' ]; then \
  echo >&2; \
  echo >&2 'error: UNIMPLEMENTED'; \
  echo >&2 'TODO install golang-any from jessie-backports for GOROOT_BOOTSTRAP (and uninstall after build)'; \
  echo >&2; \
  exit 1; \
  fi; \
  \
  export PATH="/usr/local/go/bin:$PATH"; \
  go version

ENV PATH /usr/local/go/bin:$PATH

#
#
#
#
#  888888888888                            88
#       88                                 88
#       88                                 88
#       88        ,adPPYba,    ,adPPYba,   88  ,adPPYba,
#       88       a8"     "8a  a8"     "8a  88  I8[    ""
#       88       8b       d8  8b       d8  88   `"Y8ba,
#       88       "8a,   ,a8"  "8a,   ,a8"  88  aa    ]8I
#       88        `"YbbdP"'    `"YbbdP"'   88  `"YbbdP"'
#
#

# ASCII art http://patorjk.com/software/taag/#p=display&c=bash&f=Colossal&t=Python