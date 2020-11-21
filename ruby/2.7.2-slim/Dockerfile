FROM debian:stretch-slim

RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
    bzip2 \
    ca-certificates \
    curl \
    gnupg \
    libffi-dev \
    libgmp-dev \
    libjemalloc-dev \
    libssl-dev \
    libyaml-dev \
    procps \
    wget \
    zlib1g-dev \
    ; \
    rm -rf /var/lib/apt/lists/*

# skip installing gem documentation
RUN set -eux; \
    mkdir -p /usr/local/etc; \
    { \
    echo 'install: --no-document'; \
    echo 'update: --no-document'; \
    } >> /usr/local/etc/gemrc

ENV RUBY_MAJOR 2.7
ENV RUBY_VERSION 2.7.2
ENV RUBY_DOWNLOAD_SHA256 6e5706d0d4ee4e1e2f883db9d768586b4d06567debea353c796ec45e8321c3d4
ENV RUBYGEMS_VERSION 3.1.4
ENV BUNDLER_VERSION 2.1.4

# some of ruby's build scripts are written in ruby
#   we purge system ruby later to make sure our final image uses what we just built
RUN set -eux; \
    \
    savedAptMark="$(apt-mark showmanual)"; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
    autoconf \
    bison \
    dpkg-dev \
    gcc \
    libbz2-dev \
    libgdbm-dev \
    libglib2.0-dev \
    libncurses-dev \
    libreadline-dev \
    libxml2-dev \
    libxslt-dev \
    make \
    ruby \
    wget \
    xz-utils \
    ; \
    rm -rf /var/lib/apt/lists/*; \
    \
    wget -O ruby.tar.gz "https://cache.ruby-lang.org/pub/ruby/${RUBY_MAJOR%-rc}/ruby-$RUBY_VERSION.tar.gz"; \
    echo "$RUBY_DOWNLOAD_SHA256 *ruby.tar.gz" | sha256sum --check --strict; \
    \
    mkdir -p /usr/src/ruby; \
    tar -xvzf ruby.tar.gz -C /usr/src/ruby --strip-components=1; \
    rm ruby.tar.gz; \
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
    --with-jemalloc \
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
    \
    gem update --system "$RUBYGEMS_VERSION" \
    && gem install bundler --version "$BUNDLER_VERSION" --force \
    && rm -r /root/.gem/; \
    \
    curl -sL https://deb.nodesource.com/setup_12.x | bash - \
    && apt-get install -y --no-install-recommends nodejs; \
    rm -rf /var/lib/apt/lists/*; \
    \
    # rough smoke test
    ruby --version; \
    gem --version; \
    bundle --version; \
    nodejs --version

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

CMD [ "irb" ]
