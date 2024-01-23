ARG BUILD_PLATFORM=linux/amd64
FROM node:18.12.1-bullseye-slim AS build

WORKDIR /app

COPY packages/director/ ./packages/director/
COPY packages/common/ ./packages/common
COPY packages/mongo/ ./packages/mongo
COPY packages/logger/ ./packages/logger
COPY package.json ./
COPY yarn.lock ./
COPY tsconfig.json ./

RUN apt-get update && apt-get install -y git
RUN git config --global --add safe.directory /tmp/_circleci_local_build_repo
# a few environment variables to make NPM installs easier
# good colors for most applications
ENV TERM=xterm
# avoid million NPM install messages
ENV npm_config_loglevel=warn
# allow installing when the main user is root
ENV npm_config_unsafe_perm=true

RUN npm --version \
  && npm install -g yarn@latest --force \
  && yarn --version \
  && node -p process.versions \
  && node -p 'module.paths' \
  && echo  " node version:    $(node -v) \n" \
    "npm version:     $(npm -v) \n" \
    "yarn version:    $(yarn -v) \n" \
    "debian version:  $(cat /etc/debian_version) \n" \
    "user:            $(whoami) \n"

RUN yarn install --frozen-lockfile  --network-timeout 600000

RUN yarn workspace @sorry-cypress/common build
RUN yarn workspace @sorry-cypress/mongo build
RUN yarn workspace @sorry-cypress/logger build
RUN yarn workspace @sorry-cypress/director build

RUN yarn install --production --frozen-lockfile --network-timeout 600000
RUN apt-get install -y curl && \
    curl -sf https://gobinaries.com/tj/node-prune | sh && \
    node-prune

FROM FROM node:18.12.1-bullseye-slim

ARG USER=node
WORKDIR /app

COPY --chown=node:node --from=build /app/packages/ packages/
COPY --chown=node:node --from=build /app/node_modules/ node_modules/

RUN apt-get update && apt-get install -y git
RUN git config --global --add safe.directory /tmp/_circleci_local_build_repo
RUN apt-get install -y tini
RUN apt-get install -y --upgrade bash
RUN apt-get update && \
  apt-get install --no-install-recommends -y \
  libgtk2.0-0 \
  libgtk-3-0 \
  libnotify-dev \
  libgconf-2-4 \
  libgbm-dev \
  libnss3 \
  libxss1 \
  libasound2 \
  libxtst6 \
  procps \
  xauth \
  xvfb \
  # install text editors
  vim-tiny \
  nano \
  # install emoji font
  fonts-noto-color-emoji \
  # install Chinese fonts
  # this list was copied from https://github.com/jim3ma/docker-leanote
  fonts-arphic-bkai00mp \
  fonts-arphic-bsmi00lp \
  fonts-arphic-gbsn00lp \
  fonts-arphic-gkai00mp \
  fonts-arphic-ukai \
  fonts-arphic-uming \
  ttf-wqy-zenhei \
  ttf-wqy-microhei \
  xfonts-wqy \
  # clean up
  && rm -rf /var/lib/apt/lists/* \
  && apt-get clean
ENV NODE_ENV=production

# USER ${USER}
EXPOSE 1234

ENTRYPOINT ["/usr/bin/tini", "--"]
CMD ["./node_modules/.bin/pm2-runtime", "packages/director/dist"]
