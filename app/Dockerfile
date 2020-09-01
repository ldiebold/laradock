# develop stage
FROM node:lts-alpine as develop-stage
WORKDIR /var/www/app
COPY package*.json ./
RUN yarn global add @quasar/cli
COPY . .
# build stage
FROM develop-stage as build-stage
RUN yarn
RUN quasar build
# production stage
# FROM nginx:lts-alpine as production-stage
# COPY --from=build-stage /app/dist/pwa /usr/share/nginx/html
# EXPOSE 80
# CMD ["nginx", "-g", "daemon off;"]