FROM alpine:3.12@sha256:a15790640a6690aa1730c38cf0a440e2aa44aaca9b0e8931a9f2b0d7cc90fd65

ENV NODE_VERSION 10.22.1

# good colors for most applications
ENV TERM xterm

# create new node user
RUN addgroup -g 1000 node \
    && adduser -u 1000 -G node -s /bin/sh -D node

# install deps
RUN apk add --no-cache \
        bash \
        python2 \
        libstdc++ \
        git \
        py-pip \
        py-setuptools \
        make \
        g++ \
        gcc \
        clang \
        unzip \
        curl \
        binutils-gold \
        gnupg \
        libgcc \
        linux-headers \
        tar

# switch to node user
USER node

# set home and project dirs
ENV HOME /home/node
ENV PROJECT $HOME/project

# create local bin dir
RUN mkdir -p $HOME/bin
# add local bin dir to path
ENV PATH $PATH:$HOME/bin

# create project dir
RUN mkdir -p $PROJECT

# work in local home dir
WORKDIR $HOME

# set npm path for global modules
ENV NPM_CONFIG_PREFIX $HOME/.npm-global
# add npm global packages to path
ENV PATH $PATH:$HOME/.npm-global/bin

# avoid million NPM install messages
ENV npm_config_loglevel warn

# install node
RUN ARCH= && alpineArch="$(apk --print-arch)" \
      && case "${alpineArch##*-}" in \
        x86_64) \
          ARCH='x64' \
          CHECKSUM="72f0693db768ef07c712e7a575bd6914b8a74338e91e9e969c8d7e2a832d38f3" \
          ;; \
        *) ;; \
      esac \
  && if [ -n "${CHECKSUM}" ]; then \
    set -eu; \
    curl -fsSLO --compressed "https://unofficial-builds.nodejs.org/download/release/v$NODE_VERSION/node-v$NODE_VERSION-linux-$ARCH-musl.tar.xz"; \
    echo "$CHECKSUM  node-v$NODE_VERSION-linux-$ARCH-musl.tar.xz" | sha256sum -c - \
      && tar -xJf "node-v$NODE_VERSION-linux-$ARCH-musl.tar.xz" -C $HOME --strip-components=1 --no-same-owner \
      && ln -s $HOME/bin/node $HOME/bin/nodejs; \
  else \
    echo "Building from source" \
    # gpg keys listed at https://github.com/nodejs/node#release-keys
    && for key in \
      4ED778F539E3634C779C87C6D7062848A1AB005C \
      94AE36675C464D64BAFA68DD7434390BDBE9B9C5 \
      71DCFD284A79C3B38668286BC97EC7A07EDE3FC1 \
      8FCCA13FEF1D0C2E91008E09770F7A9A5AE15600 \
      C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
      C82FA3AE1CBEDC6BE46B9360C43CEC45C17AB93C \
      DD8F2338BAE7501E3DD5AC78C273792F7D83545D \
      A48C2BEE680E841632CD4E44F07496B3EB3C1762 \
      108F52B48DB57BB0CC439B2997B01419BD92F80A \
      B9E2F5981AA6E0CD28160D9FF13993A75599653C \
    ; do \
      gpg --batch --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" || \
      gpg --batch --keyserver hkp://ipv4.pool.sks-keyservers.net --recv-keys "$key" || \
      gpg --batch --keyserver hkp://pgp.mit.edu:80 --recv-keys "$key" ; \
    done \
    && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION.tar.xz" \
    && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
    && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
    && grep " node-v$NODE_VERSION.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
    && tar -xf "node-v$NODE_VERSION.tar.xz" \
    && cd "node-v$NODE_VERSION" \
    && ./configure \
    && make -j$(getconf _NPROCESSORS_ONLN) V= \
    && make install \
    && cd .. \
    && rm -Rf "node-v$NODE_VERSION" \
    && rm "node-v$NODE_VERSION.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt; \
  fi \
  && rm -f "node-v$NODE_VERSION-linux-$ARCH-musl.tar.xz"

# install latest npm, regardless of node version, for speed and fixes
RUN npm install -g npm@6.14.8
# remove old npm
RUN rm $HOME/bin/npm
# verify npm works
RUN npm doctor

# install yarn
RUN for key in \
    6A010C5166006599AA17F08146C2130DFD2497F5 \
  ; do \
    gpg --batch --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" || \
    gpg --batch --keyserver hkp://ipv4.pool.sks-keyservers.net --recv-keys "$key" || \
    gpg --batch --keyserver hkp://pgp.mit.edu:80 --recv-keys "$key" ; \
  done \
  && curl -fsSLO --compressed "https://yarnpkg.com/downloads/1.22.10/yarn-v1.22.10.tar.gz" \
  && curl -fsSLO --compressed "https://yarnpkg.com/downloads/1.22.10/yarn-v1.22.10.tar.gz.asc" \
  && gpg --batch --verify yarn-v1.22.10.tar.gz.asc yarn-v1.22.10.tar.gz \
  && mkdir -p $HOME \
  && tar -xzf yarn-v1.22.10.tar.gz -C $HOME \
  && ln -s $HOME/yarn-v1.22.10/bin/yarn $HOME/bin/yarn \
  && ln -s $HOME/yarn-v1.22.10/bin/yarnpkg $HOME/bin/yarnpkg \
  && rm yarn-v1.22.10.tar.gz.asc yarn-v1.22.10.tar.gz

# add yarn global packages to path
ENV PATH $PATH:$HOME/.yarn/bin:$HOME/.config/yarn/global/node_modules/.bin

# set yarn path for global modules
RUN yarn config set prefix $HOME/.yarn

COPY docker-entrypoint.sh $HOME/bin/
ENTRYPOINT ["docker-entrypoint.sh"]

WORKDIR $PROJECT

# print versions of local tools
RUN echo "node version:  $(node -v) \n" \
  "npm version:           $(npm -v) \n" \
  "yarn version:          $(yarn -v) \n" \
  "git version:           $(git --version) \n" \
  "user:                  $(whoami) \n"

CMD ["node"]
