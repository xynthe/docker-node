FROM cimg/node:12.19@sha256:d68557a8c20fbde71451e5394b6963fc9ad70a9a58faa104393908ca2c7ca75c

# home folder
ENV HOME /home/circleci

# good colors for most applications
ENV TERM xterm

# no interactive frontend during docker build
ENV DEBIAN_FRONTEND noninteractive
ENV DEBCONF_NONINTERACTIVE_SEEN true

# set npm path for global modules
ENV NPM_CONFIG_PREFIX ${HOME}/.npm-global

# create local bin dir
RUN mkdir ${HOME}/bin

# add npm global packages & local bin dir to path
ENV PATH $PATH:${HOME}/.npm-global/bin:${HOME}/bin

# avoid million NPM install messages
ENV npm_config_loglevel warn

# install latest npm, regardless of node version, for speed and fixes
RUN npm i npm@6.14.8 -g

WORKDIR ${HOME}/project
