FROM cypress/base

USER root
RUN npm install -g @quasar/cli

RUN mkdir /home/node/app

# VOLUME [ "/home/node/app" ]
WORKDIR /home/node/app

CMD /bin/sh