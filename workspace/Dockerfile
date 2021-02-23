#
#--------------------------------------------------------------------------
# Image Setup
#--------------------------------------------------------------------------
#
# To edit the 'workspace' base Image, visit its repository on Github
#    https://github.com/Laradock/workspace
#
# To change its version, see the available Tags on the Docker Hub:
#    https://hub.docker.com/r/laradock/workspace/tags/
#
# Note: Base Image name format {image-tag}-{php-version}
#

ARG LARADOCK_PHP_VERSION
ARG BASE_IMAGE_TAG_PREFIX=latest
FROM laradock/workspace:${BASE_IMAGE_TAG_PREFIX}-${LARADOCK_PHP_VERSION}

LABEL maintainer="Mahmoud Zalt <mahmoud@zalt.me>"

ARG LARADOCK_PHP_VERSION

# Set Environment Variables
ENV DEBIAN_FRONTEND noninteractive

# Start as root
USER root

###########################################################################
# Laradock non-root user:
###########################################################################

# Add a non-root user to prevent files being created with root permissions on host machine.
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

ARG CHANGE_SOURCE=false
ARG UBUNTU_SOURCE
COPY ./sources.sh /tmp/sources.sh

RUN if [ ${CHANGE_SOURCE} = true ]; then \
  chmod +x /tmp/sources.sh && \
  /bin/sh -c /tmp/sources.sh && \
  rm -rf /tmp/sources.sh \
  ;fi

# always run apt update when start and after add new source list, then clean up at end.
RUN set -xe; \
  apt-get update -yqq && \
  pecl channel-update pecl.php.net && \
  groupadd -g ${PGID} laradock && \
  useradd -u ${PUID} -g laradock -m laradock -G docker_env && \
  usermod -p "*" laradock -s /bin/bash && \
  apt-get install -yqq \
  apt-utils \
  #
  #--------------------------------------------------------------------------
  # Mandatory Software's Installation
  #--------------------------------------------------------------------------
  #
  # Mandatory Software's such as ("php-cli", "git", "vim", ....) are
  # installed on the base image 'laradock/workspace' image. If you want
  # to add more Software's or remove existing one, you need to edit the
  # base image (https://github.com/Laradock/workspace).
  #
  # next lines are here because there is no auto build on dockerhub see https://github.com/laradock/laradock/pull/1903#issuecomment-463142846
  libzip-dev zip unzip \
  # Install the zip extension
  php${LARADOCK_PHP_VERSION}-zip \
  # nasm
  nasm && \
  php -m | grep -q 'zip'

#
#--------------------------------------------------------------------------
# Optional Software's Installation
#--------------------------------------------------------------------------
#
# Optional Software's will only be installed if you set them to `true`
# in the `docker-compose.yml` before the build.
# Example:
#   - INSTALL_NODE=false
#   - ...
#

###########################################################################
# Set Timezone
###########################################################################

ARG TZ=UTC
ENV TZ ${TZ}

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

###########################################################################
# User Aliases
###########################################################################

USER root

COPY ./aliases.sh /root/aliases.sh
COPY ./aliases.sh /home/laradock/aliases.sh

RUN sed -i 's/\r//' /root/aliases.sh && \
  sed -i 's/\r//' /home/laradock/aliases.sh && \
  chown laradock:laradock /home/laradock/aliases.sh && \
  echo "" >> ~/.bashrc && \
  echo "# Load Custom Aliases" >> ~/.bashrc && \
  echo "source ~/aliases.sh" >> ~/.bashrc && \
  echo "" >> ~/.bashrc

USER laradock

RUN echo "" >> ~/.bashrc && \
  echo "# Load Custom Aliases" >> ~/.bashrc && \
  echo "source ~/aliases.sh" >> ~/.bashrc && \
  echo "" >> ~/.bashrc

###########################################################################
# Composer:
###########################################################################

USER root

# Add the composer.json
COPY ./composer.json /home/laradock/.composer/composer.json

# Add the auth.json for magento 2 credentials
COPY ./auth.json /home/laradock/.composer/auth.json

# Make sure that ~/.composer belongs to laradock
RUN chown -R laradock:laradock /home/laradock/.composer

# Export composer vendor path
RUN echo "" >> ~/.bashrc && \
  echo 'export PATH="$HOME/.composer/vendor/bin:$PATH"' >> ~/.bashrc

USER laradock

# Check if global install need to be ran
ARG COMPOSER_GLOBAL_INSTALL=false
ENV COMPOSER_GLOBAL_INSTALL ${COMPOSER_GLOBAL_INSTALL}

RUN if [ ${COMPOSER_GLOBAL_INSTALL} = true ]; then \
  # run the install
  composer global install \
  ;fi

# Check if auth file is disabled
ARG COMPOSER_AUTH=false
ENV COMPOSER_AUTH ${COMPOSER_AUTH}

RUN if [ ${COMPOSER_AUTH} = false ]; then \
  # remove the file
  rm /home/laradock/.composer/auth.json \
  ;fi

ARG COMPOSER_REPO_PACKAGIST
ENV COMPOSER_REPO_PACKAGIST ${COMPOSER_REPO_PACKAGIST}

RUN if [ ${COMPOSER_REPO_PACKAGIST} ]; then \
  composer config -g repo.packagist composer ${COMPOSER_REPO_PACKAGIST} \
  ;fi

# Export composer vendor path
RUN echo "" >> ~/.bashrc && \
  echo 'export PATH="~/.composer/vendor/bin:$PATH"' >> ~/.bashrc

###########################################################################
# Non-root user : PHPUnit path
###########################################################################

# add ./vendor/bin to non-root user's bashrc (needed for phpunit)
USER laradock

RUN echo "" >> ~/.bashrc && \
  echo 'export PATH="/var/www/vendor/bin:$PATH"' >> ~/.bashrc

###########################################################################
# Crontab
###########################################################################

USER root

COPY ./crontab /etc/cron.d

RUN chmod -R 644 /etc/cron.d

###########################################################################
# GMP (GNU Multiple Precision):
###########################################################################

USER root

ARG INSTALL_GMP=false
ARG PHP_VERSION=${LARADOCK_PHP_VERSION}

RUN if [ ${INSTALL_GMP} = true ]; then \
  # Install the PHP GMP extension
  apt-get -y install php${LARADOCK_PHP_VERSION}-gmp \
  ;fi

###########################################################################
# SSH2:
###########################################################################

USER root

ARG INSTALL_SSH2=false

RUN if [ ${INSTALL_SSH2} = true ]; then \
  # Install the PHP SSH2 extension
  apt-get -y install libssh2-1-dev php${LARADOCK_PHP_VERSION}-ssh2 \
  ;fi

###########################################################################
# SOAP:
###########################################################################

USER root

ARG INSTALL_SOAP=false

RUN if [ ${INSTALL_SOAP} = true ]; then \
  # Install the PHP SOAP extension
  apt-get -y install libxml2-dev php${LARADOCK_PHP_VERSION}-soap \
  ;fi

###########################################################################
# XSL:
###########################################################################

USER root

ARG INSTALL_XSL=false

RUN if [ ${INSTALL_XSL} = true ]; then \
  # Install the PHP XSL extension
  apt-get -y install libxslt-dev php${LARADOCK_PHP_VERSION}-xsl \
  ;fi


###########################################################################
# LDAP:
###########################################################################

ARG INSTALL_LDAP=false

RUN if [ ${INSTALL_LDAP} = true ]; then \
  apt-get install -y libldap2-dev && \
  apt-get install -y php${LARADOCK_PHP_VERSION}-ldap \
  ;fi

###########################################################################
# SMB:
###########################################################################

ARG INSTALL_SMB=false

RUN if [ ${INSTALL_SMB} = true ]; then \
  apt-get install -y smbclient php-smbclient coreutils \
  ;fi

###########################################################################
# IMAP:
###########################################################################

ARG INSTALL_IMAP=false

RUN if [ ${INSTALL_IMAP} = true ]; then \
  apt-get install -y php${LARADOCK_PHP_VERSION}-imap \
  ;fi

###########################################################################
# Subversion:
###########################################################################

USER root

ARG INSTALL_SUBVERSION=false

RUN if [ ${INSTALL_SUBVERSION} = true ]; then \
  apt-get install -y subversion \
  ;fi


###########################################################################
# xDebug:
###########################################################################

USER root

ARG INSTALL_XDEBUG=false

RUN if [ ${INSTALL_XDEBUG} = true ]; then \
  # Load the xdebug extension only with phpunit commands
  apt-get install -y php${LARADOCK_PHP_VERSION}-xdebug && \
  sed -i 's/^;//g' /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/20-xdebug.ini \
  ;fi

# ADD for REMOTE debugging
COPY ./xdebug.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/xdebug.ini

RUN sed -i "s/xdebug.remote_autostart=0/xdebug.remote_autostart=1/" /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/xdebug.ini && \
  sed -i "s/xdebug.remote_enable=0/xdebug.remote_enable=1/" /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/xdebug.ini && \
  sed -i "s/xdebug.cli_color=0/xdebug.cli_color=1/" /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/xdebug.ini

###########################################################################
# pcov:
###########################################################################

USER root

ARG INSTALL_PCOV=false

RUN if [ ${INSTALL_PCOV} = true ]; then \
  if [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ]; then \
  if [ $(php -r "echo PHP_MINOR_VERSION;") != "0" ]; then \
  pecl install pcov && \
  echo "extension=pcov.so" >> /etc/php/${LARADOCK_PHP_VERSION}/cli/php.ini && \
  echo "pcov.enabled" >> /etc/php/${LARADOCK_PHP_VERSION}/cli/php.ini \
  ;fi \
  ;fi \
  ;fi


###########################################################################
# Phpdbg:
###########################################################################

USER root

ARG INSTALL_PHPDBG=false

RUN if [ ${INSTALL_PHPDBG} = true ]; then \
  # Load the xdebug extension only with phpunit commands
  apt-get install -y --force-yes php${LARADOCK_PHP_VERSION}-phpdbg \
  ;fi

###########################################################################
# ssh:
###########################################################################

ARG INSTALL_WORKSPACE_SSH=false

COPY insecure_id_rsa /tmp/id_rsa
COPY insecure_id_rsa.pub /tmp/id_rsa.pub

RUN if [ ${INSTALL_WORKSPACE_SSH} = true ]; then \
  rm -f /etc/service/sshd/down && \
  cat /tmp/id_rsa.pub >> /root/.ssh/authorized_keys \
  && cat /tmp/id_rsa.pub >> /root/.ssh/id_rsa.pub \
  && cat /tmp/id_rsa >> /root/.ssh/id_rsa \
  && rm -f /tmp/id_rsa* \
  && chmod 644 /root/.ssh/authorized_keys /root/.ssh/id_rsa.pub \
  && chmod 400 /root/.ssh/id_rsa \
  && cp -rf /root/.ssh /home/laradock \
  && chown -R laradock:laradock /home/laradock/.ssh \
  ;fi

###########################################################################
# PHP REDIS EXTENSION
###########################################################################

ARG INSTALL_PHPREDIS=false

RUN if [ ${INSTALL_PHPREDIS} = true ]; then \
  apt-get update -yqq && \
  apt-get install -yqq php${LARADOCK_PHP_VERSION}-redis \
  ;fi

###########################################################################
# Libpng16 EXTENSION
###########################################################################

ARG INSTALL_LIBPNG=false

RUN if [ ${INSTALL_LIBPNG} = true ]; then \
  apt-get update && \
  apt-get install libpng16-16 \
  ;fi

###########################################################################
# Inotify EXTENSION:
###########################################################################

ARG INSTALL_INOTIFY=false

RUN if [ ${INSTALL_INOTIFY} = true ]; then \
  pecl -q install inotify && \
  echo "extension=inotify.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/inotify.ini && \
  ln -s /etc/php/${LARADOCK_PHP_VERSION}/mods-available/inotify.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/20-inotify.ini \
  ;fi

###########################################################################
# AST EXTENSION
###########################################################################

ARG INSTALL_AST=false
ARG AST_VERSION=1.0.4
ENV AST_VERSION ${AST_VERSION}

RUN if [ ${INSTALL_AST} = true ]; then \
  # AST extension requires PHP 7.0.0 or newer
  if [ $(php -r "echo PHP_MAJOR_VERSION;") != "5" ]; then \
  # Install AST extension
  printf "\n" | pecl -q install ast-${AST_VERSION} && \
  echo "extension=ast.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/ast.ini && \
  phpenmod -v ${LARADOCK_PHP_VERSION} -s cli ast \
  ;fi \
  ;fi

###########################################################################
# fswatch
###########################################################################

ARG INSTALL_FSWATCH=false

RUN if [ ${INSTALL_FSWATCH} = true ]; then \
  apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 47FE03C1 \
  && add-apt-repository -y ppa:hadret/fswatch \
  || apt-get update -yqq \
  && apt-get -y install fswatch \
  ;fi

###########################################################################
# Node / NVM:
###########################################################################

# Check if NVM needs to be installed
ARG NODE_VERSION=node
ENV NODE_VERSION ${NODE_VERSION}
ARG INSTALL_NODE=false
ARG INSTALL_LERNA=false
ARG INSTALL_NPM_GULP=false
ARG INSTALL_NPM_BOWER=false
ARG INSTALL_NPM_VUE_CLI=false
ARG INSTALL_NPM_ANGULAR_CLI=false
ARG NPM_REGISTRY
ENV NPM_REGISTRY ${NPM_REGISTRY}
ENV NVM_DIR /home/laradock/.nvm
ARG NVM_NODEJS_ORG_MIRROR
ENV NVM_NODEJS_ORG_MIRROR ${NVM_NODEJS_ORG_MIRROR}

RUN if [ ${INSTALL_NODE} = true ]; then \
  # Install nvm (A Node Version Manager)
  mkdir -p $NVM_DIR && \
  curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash \
  && . $NVM_DIR/nvm.sh \
  && nvm install ${NODE_VERSION} \
  && nvm use ${NODE_VERSION} \
  && nvm alias ${NODE_VERSION} \
  && if [ ${NPM_REGISTRY} ]; then \
  npm config set registry ${NPM_REGISTRY} \
  ;fi \
  && if [ ${INSTALL_NPM_GULP} = true ]; then \
  npm install -g gulp \
  ;fi \
  && if [ ${INSTALL_LERNA} = true ]; then \
  npm install -g lerna \
  ;fi \
  && if [ ${INSTALL_NPM_BOWER} = true ]; then \
  npm install -g bower \
  ;fi \
  && if [ ${INSTALL_NPM_VUE_CLI} = true ]; then \
  npm install -g @vue/cli \
  ;fi \
  && if [ ${INSTALL_NPM_ANGULAR_CLI} = true ]; then \
  npm install -g @angular/cli \
  ;fi \
  && ln -s `npm bin --global` /home/laradock/.node-bin \
  ;fi

# Wouldn't execute when added to the RUN statement in the above block
# Source NVM when loading bash since ~/.profile isn't loaded on non-login shell
RUN if [ ${INSTALL_NODE} = true ]; then \
  echo "" >> ~/.bashrc && \
  echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.bashrc && \
  echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc \
  ;fi

# Add NVM binaries to root's .bashrc
USER root

RUN if [ ${INSTALL_NODE} = true ]; then \
  echo "" >> ~/.bashrc && \
  echo 'export NVM_DIR="/home/laradock/.nvm"' >> ~/.bashrc && \
  echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc \
  ;fi

# Add PATH for node
ENV PATH $PATH:/home/laradock/.node-bin

# Make it so the node modules can be executed with 'docker-compose exec'
# We'll create symbolic links into '/usr/local/bin'.
RUN if [ ${INSTALL_NODE} = true ]; then \
  find $NVM_DIR -type f -name node -exec ln -s {} /usr/local/bin/node \; && \
  NODE_MODS_DIR="$NVM_DIR/versions/node/$(node -v)/lib/node_modules" && \
  ln -s $NODE_MODS_DIR/bower/bin/bower /usr/local/bin/bower && \
  ln -s $NODE_MODS_DIR/gulp/bin/gulp.js /usr/local/bin/gulp && \
  ln -s $NODE_MODS_DIR/npm/bin/npm-cli.js /usr/local/bin/npm && \
  ln -s $NODE_MODS_DIR/npm/bin/npx-cli.js /usr/local/bin/npx && \
  ln -s $NODE_MODS_DIR/vue-cli/bin/vue /usr/local/bin/vue && \
  ln -s $NODE_MODS_DIR/vue-cli/bin/vue-init /usr/local/bin/vue-init && \
  ln -s $NODE_MODS_DIR/vue-cli/bin/vue-list /usr/local/bin/vue-list \
  ;fi

RUN if [ ${NPM_REGISTRY} ]; then \
  . ~/.bashrc && npm config set registry ${NPM_REGISTRY} \
  ;fi

###########################################################################
# YARN:
###########################################################################

USER laradock

ARG INSTALL_YARN=false
ARG YARN_VERSION=latest
ENV YARN_VERSION ${YARN_VERSION}

RUN if [ ${INSTALL_YARN} = true ]; then \
  [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" && \
  if [ ${YARN_VERSION} = "latest" ]; then \
  curl -o- -L https://yarnpkg.com/install.sh | bash; \
  else \
  curl -o- -L https://yarnpkg.com/install.sh | bash -s -- --version ${YARN_VERSION}; \
  fi && \
  echo "" >> ~/.bashrc && \
  echo 'export PATH="$HOME/.yarn/bin:$PATH"' >> ~/.bashrc \
  ;fi

# Add YARN binaries to root's .bashrc
USER root

RUN if [ ${INSTALL_YARN} = true ]; then \
  echo "" >> ~/.bashrc && \
  echo 'export YARN_DIR="/home/laradock/.yarn"' >> ~/.bashrc && \
  echo 'export PATH="$YARN_DIR/bin:$PATH"' >> ~/.bashrc \
  ;fi

# Add PATH for YARN
ENV PATH $PATH:/home/laradock/.yarn/bin

###########################################################################
# Linuxbrew:
###########################################################################

USER root

ARG INSTALL_LINUXBREW=false

RUN if [ ${INSTALL_LINUXBREW} = true ]; then \
  # Preparation
  apt-get upgrade -y && \
  apt-get install -y build-essential make cmake scons curl git \
  ruby autoconf automake autoconf-archive \
  gettext libtool flex bison \
  libbz2-dev libcurl4-openssl-dev \
  libexpat-dev libncurses-dev && \
  # Install the Linuxbrew
  git clone --depth=1 https://github.com/Homebrew/linuxbrew.git ~/.linuxbrew && \
  echo "" >> ~/.bashrc && \
  echo 'export PKG_CONFIG_PATH"=/usr/local/lib/pkgconfig:/usr/local/lib64/pkgconfig:/usr/lib64/pkgconfig:/usr/lib/pkgconfig:/usr/lib/x86_64-linux-gnu/pkgconfig:/usr/lib64/pkgconfig:/usr/share/pkgconfig:$PKG_CONFIG_PATH"' >> ~/.bashrc && \
  # Setup linuxbrew
  echo 'export LINUXBREWHOME="$HOME/.linuxbrew"' >> ~/.bashrc && \
  echo 'export PATH="$LINUXBREWHOME/bin:$PATH"' >> ~/.bashrc && \
  echo 'export MANPATH="$LINUXBREWHOME/man:$MANPATH"' >> ~/.bashrc && \
  echo 'export PKG_CONFIG_PATH="$LINUXBREWHOME/lib64/pkgconfig:$LINUXBREWHOME/lib/pkgconfig:$PKG_CONFIG_PATH"' >> ~/.bashrc && \
  echo 'export LD_LIBRARY_PATH="$LINUXBREWHOME/lib64:$LINUXBREWHOME/lib:$LD_LIBRARY_PATH"' >> ~/.bashrc \
  ;fi

###########################################################################
# PYTHON2:
###########################################################################

ARG INSTALL_PYTHON=true

RUN if [ ${INSTALL_PYTHON} = true ]; then \
  apt-get update -yqq \
  && apt-get -y install python python-pip python-dev build-essential  \
  && python -m pip install --upgrade pip  \
  && python -m pip install --upgrade virtualenv \
  ;fi

###########################################################################
# PYTHON3:
###########################################################################

ARG INSTALL_PYTHON3=false

RUN if [ ${INSTALL_PYTHON3} = true ]; then \
  apt-get -y install python3 python3-pip python3-dev build-essential  \
  && python3 -m pip install --upgrade --force-reinstall pip  \
  && python3 -m pip install --upgrade virtualenv \
  ;fi

###########################################################################
# SUPERVISOR:
###########################################################################
ARG INSTALL_SUPERVISOR=false

RUN if [ ${INSTALL_SUPERVISOR} = true ]; then \
  if [ ${INSTALL_PYTHON} = true ]; then \
  python -m pip install --upgrade supervisor && \
  echo_supervisord_conf > /etc/supervisord.conf && \
  sed -i 's/\;\[include\]/\[include\]/g' /etc/supervisord.conf && \
  sed -i 's/\;files\s.*/files = supervisord.d\/*.conf/g' /etc/supervisord.conf \
  ;fi \
  ;fi

USER laradock

###########################################################################
# MySQL Client:
###########################################################################

USER root

ARG INSTALL_MYSQL_CLIENT=false

RUN if [ ${INSTALL_MYSQL_CLIENT} = true ]; then \
  apt-get update -yqq && \
  apt-get -y install mysql-client \
  ;fi

###########################################################################
# ping:
###########################################################################

USER root

ARG INSTALL_PING=false

RUN if [ ${INSTALL_PING} = true ]; then \
  apt-get update -yqq && \
  apt-get -y install inetutils-ping \
  ;fi

###########################################################################
# sshpass:
###########################################################################

USER root

ARG INSTALL_SSHPASS=false

RUN if [ ${INSTALL_SSHPASS} = true ]; then \
  apt-get update -yqq && \
  apt-get -y install sshpass \
  ;fi

###########################################################################
# YAML: extension for PHP-CLI
###########################################################################

USER root

ARG INSTALL_YAML=false

RUN if [ ${INSTALL_YAML} = true ]; then \
  apt-get install libyaml-dev -y ; \
  if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
  pecl install -a yaml-1.3.2; \
  else \
  pecl install yaml; \
  fi && \
  echo "extension=yaml.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/yaml.ini && \
  ln -s /etc/php/${LARADOCK_PHP_VERSION}/mods-available/yaml.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/35-yaml.ini \
  ;fi

###########################################################################
# Bash Git Prompt
###########################################################################

ARG INSTALL_GIT_PROMPT=false

COPY git-prompt.sh /tmp/git-prompt

RUN if [ ${INSTALL_GIT_PROMPT} = true ]; then \
  git clone https://github.com/magicmonty/bash-git-prompt.git /root/.bash-git-prompt --depth=1 && \
  cat /tmp/git-prompt >> /root/.bashrc && \
  rm /tmp/git-prompt \
  ;fi

###########################################################################
# XMLRPC:
###########################################################################

ARG INSTALL_XMLRPC=false

RUN if [ ${INSTALL_XMLRPC} = true ]; then \
  docker-php-ext-install xmlrpc \
  ;fi

###########################################################################
# Check PHP version:
###########################################################################

RUN set -xe; php -v | head -n 1 | grep -q "PHP ${LARADOCK_PHP_VERSION}."

###########################################################################
# SSL Certificates
###########################################################################

COPY ssl/agripath-authority.crt /usr/local/share/ca-certificates
COPY ssl/wildcard.agripath.test.crt /usr/local/share/ca-certificates
COPY ssl/wildcard.agripath.test.key /usr/local/share/ca-certificates
RUN dpkg-reconfigure ca-certificates && update-ca-certificates

###########################################################################
# Oh My ZSH!
###########################################################################

USER root

ARG SHELL_OH_MY_ZSH=false
RUN if [ ${SHELL_OH_MY_ZSH} = true ]; then \
  apt install -y zsh \
  ;fi

USER laradock
RUN if [ ${SHELL_OH_MY_ZSH} = true ]; then \
  sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh) --keep-zshrc" && \
  sed -i -r 's/^plugins=\(.*?\)$/plugins=(laravel5)/' /home/laradock/.zshrc && \
  echo '\n\
  bindkey "^[OB" down-line-or-search\n\
  bindkey "^[OC" forward-char\n\
  bindkey "^[OD" backward-char\n\
  bindkey "^[OF" end-of-line\n\
  bindkey "^[OH" beginning-of-line\n\
  bindkey "^[[1~" beginning-of-line\n\
  bindkey "^[[3~" delete-char\n\
  bindkey "^[[4~" end-of-line\n\
  bindkey "^[[5~" up-line-or-history\n\
  bindkey "^[[6~" down-line-or-history\n\
  bindkey "^?" backward-delete-char\n' >> /home/laradock/.zshrc \
  ;fi

USER root

###########################################################################
# Cypress Dependencies
###########################################################################

RUN apt-get update && apt-get install --no-install-recommends -y curl \
  && curl -L https://dl.google.com/linux/linux_signing_key.pub | apt-key add - \
  && echo "deb http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google.list \
  && apt-get update \
  && apt-get -y install libgtk2.0-0 libgtk-3-0 libgbm-dev libnotify-dev libgconf-2-4 libnss3 libxss1 libasound2 libxtst6 xauth xvfb dbus-x11 libcanberra-gtk-module libcanberra-gtk3-module google-chrome-stable

###########################################################################
# NPM:
###########################################################################

COPY .npmrc /root

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

USER root

# Clean up
RUN apt-get clean && \
  rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
  rm /var/log/lastlog /var/log/faillog

# Set default work directory
WORKDIR /var/www
