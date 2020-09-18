# specify the node base image with your desired version node:<version>
FROM node:10

COPY .npmrc /root

RUN cd /var/www/app \
    && npm config set '//registry.npmjs.org/:_authToken' "${NPM_TOKEN}" \
    && npm publish \
    && yarn add @quasar/cli \
    && yarn