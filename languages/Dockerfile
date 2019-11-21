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