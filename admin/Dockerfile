# specify the node base image with your desired version node:<version>
FROM node:10

COPY .npmrc /root

RUN yarn add @quasar/cli \
    && yarn