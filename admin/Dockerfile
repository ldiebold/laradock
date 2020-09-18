# specify the node base image with your desired version node:<version>
FROM node:10

RUN npm config set '//registry.npmjs.org/:_authToken' "${NPM_TOKEN}" \
    && npm publish