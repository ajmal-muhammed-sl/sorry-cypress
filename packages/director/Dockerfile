ARG BUILD_PLATFORM=linux/amd64
FROM cypress/browsers:node18.12.0-chrome107 AS build

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
RUN yarn install --frozen-lockfile  --network-timeout 600000

RUN yarn workspace @sorry-cypress/common build
RUN yarn workspace @sorry-cypress/mongo build
RUN yarn workspace @sorry-cypress/logger build
RUN yarn workspace @sorry-cypress/director build

RUN yarn install --production --frozen-lockfile --network-timeout 600000
RUN apt-get install -y curl && \
    curl -sf https://gobinaries.com/tj/node-prune | sh && \
    node-prune

FROM cypress/browsers:node18.12.0-chrome107

ARG USER=node
WORKDIR /app

COPY --chown=node:node --from=build /app/packages/ packages/
COPY --chown=node:node --from=build /app/node_modules/ node_modules/

RUN apt-get update && apt-get install -y git
RUN git config --global --add safe.directory /tmp/_circleci_local_build_repo
RUN apt-get install -y tini
RUN apt-get install -y --upgrade bash
RUN apt-get install libgtk2.0-0 libgtk-3-0 libgbm-dev libnotify-dev libnss3 libxss1 libasound2 libxtst6 xauth xvfb
ENV NODE_ENV=production

# USER ${USER}
EXPOSE 1234

ENTRYPOINT ["/usr/bin/tini", "--"]
CMD ["./node_modules/.bin/pm2-runtime", "packages/director/dist"]
