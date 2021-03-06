FROM buildpack-deps:focal@sha256:1967396d9c9e37ff2aa2c165e61e77394720250e5a8d8435e07c1a7b586378fa

ENV NODE_VERSION 10.22.1

# good colors for most applications
ENV TERM xterm

# no interactive frontend during docker build
ENV DEBIAN_FRONTEND noninteractive
ENV DEBCONF_NONINTERACTIVE_SEEN true

# create new node user
RUN groupadd --gid 1000 node \
  && useradd --uid 1000 --gid node --shell /bin/bash --create-home node

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
RUN ARCH= && dpkgArch="$(dpkg --print-architecture)" \
  && case "${dpkgArch##*-}" in \
    amd64) ARCH='x64';; \
    ppc64el) ARCH='ppc64le';; \
    s390x) ARCH='s390x';; \
    arm64) ARCH='arm64';; \
    armhf) ARCH='armv7l';; \
    i386) ARCH='x86';; \
    *) echo "unsupported architecture"; exit 1 ;; \
  esac \
  # gpg keys listed at https://github.com/nodejs/node#release-keys
  && set -ex \
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
  && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-$ARCH.tar.xz" \
  && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
  && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
  && grep " node-v$NODE_VERSION-linux-$ARCH.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
  && tar -xJf "node-v$NODE_VERSION-linux-$ARCH.tar.xz" -C $HOME --strip-components=1 --no-same-owner \
  && rm "node-v$NODE_VERSION-linux-$ARCH.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
  && ln -s $HOME/bin/node $HOME/bin/nodejs

# install latest npm, regardless of node version, for speed and fixes
RUN npm install -g npm@6.14.8
# remove old npm
RUN rm $HOME/bin/npm
# verify npm works
RUN npm doctor

# install yarn
RUN set -ex \
  && for key in \
    6A010C5166006599AA17F08146C2130DFD2497F5 \
  ; do \
    gpg --batch --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" || \
    gpg --batch --keyserver hkp://ipv4.pool.sks-keyservers.net --recv-keys "$key" || \
    gpg --batch --keyserver hkp://pgp.mit.edu:80 --recv-keys "$key" ; \
  done \
  && curl -fsSLO --compressed "https://yarnpkg.com/downloads/1.22.10/yarn-v1.22.10.tar.gz" \
  && curl -fsSLO --compressed "https://yarnpkg.com/downloads/1.22.10/yarn-v1.22.10.tar.gz.asc" \
  && gpg --batch --verify yarn-v1.22.10.tar.gz.asc yarn-v1.22.10.tar.gz \
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
  "user:                  $(whoami) \n" \
  "ubuntu version:        $(cat /etc/lsb-release)"

CMD ["node"]
