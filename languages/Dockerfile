FROM brandonkal/docker-polyglot:base

###############################################################################
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
# Taken From:      d8'
# https://github.com/docker-library/python/blob/master/3.7/stretch/Dockerfile
# Modifications:
#
# Changed FROM to be compatible with `buildfrom.sh`
# Removed CMD ["python"] at the end.
# Added DEBIAN_FRONTEND=noninteractive to apt-install to prevent prompt.

# ensure local python is preferred over distribution python
ENV PATH /usr/local/bin:$PATH

# http://bugs.python.org/issue19846
# > At the moment, setting "LANG=C" on a Linux system *fundamentally breaks Python 3*, and that's not OK.
ENV LANG C.UTF-8

# extra dependencies (over what buildpack-deps already includes)
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
  tk-dev \
  uuid-dev \
  && rm -rf /var/lib/apt/lists/*

ENV GPG_KEY 0D96DF4D4110E5C43FBFB17F2D347EA6AA65421D
ENV PYTHON_VERSION 3.7.5

RUN set -ex \
  \
  && wget -O python.tar.xz "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz" \
  && wget -O python.tar.xz.asc "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz.asc" \
  && export GNUPGHOME="$(mktemp -d)" \
  && gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys "$GPG_KEY" \
  && gpg --batch --verify python.tar.xz.asc python.tar.xz \
  && { command -v gpgconf > /dev/null && gpgconf --kill all || :; } \
  && rm -rf "$GNUPGHOME" python.tar.xz.asc \
  && mkdir -p /usr/src/python \
  && tar -xJC /usr/src/python --strip-components=1 -f python.tar.xz \
  && rm python.tar.xz \
  \
  && cd /usr/src/python \
  && gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
  && ./configure \
  --build="$gnuArch" \
  --enable-loadable-sqlite-extensions \
  --enable-optimizations \
  --enable-shared \
  --with-system-expat \
  --with-system-ffi \
  --without-ensurepip \
  && make -j "$(nproc)" \
  # setting PROFILE_TASK makes "--enable-optimizations" reasonable: https://bugs.python.org/issue36044 / https://github.com/docker-library/python/issues/160#issuecomment-509426916
  PROFILE_TASK='-m test.regrtest --pgo \
  test_array \
  test_base64 \
  test_binascii \
  test_binhex \
  test_binop \
  test_bytes \
  test_c_locale_coercion \
  test_class \
  test_cmath \
  test_codecs \
  test_compile \
  test_complex \
  test_csv \
  test_decimal \
  test_dict \
  test_float \
  test_fstring \
  test_hashlib \
  test_io \
  test_iter \
  test_json \
  test_long \
  test_math \
  test_memoryview \
  test_pickle \
  test_re \
  test_set \
  test_slice \
  test_struct \
  test_threading \
  test_time \
  test_traceback \
  test_unicode \
  ' \
  && make install \
  && ldconfig \
  \
  && find /usr/local -depth \
  \( \
  \( -type d -a \( -name test -o -name tests -o -name idle_test \) \) \
  -o \
  \( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
  \) -exec rm -rf '{}' + \
  && rm -rf /usr/src/python \
  \
  && python3 --version

# make some useful symlinks that are expected to exist
RUN cd /usr/local/bin \
  && ln -s idle3 idle \
  && ln -s pydoc3 pydoc \
  && ln -s python3 python \
  && ln -s python3-config python-config

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
#
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
#
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
#
#
#  88888888ba   88        88  88888888ba
#  88      "8b  88        88  88      "8b
#  88      ,8P  88        88  88      ,8P
#  88aaaaaa8P'  88aaaaaaaa88  88aaaaaa8P'
#  88""""""'    88""""""""88  88""""""'
#  88           88        88  88
#  88           88        88  88
#  88           88        88  88
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