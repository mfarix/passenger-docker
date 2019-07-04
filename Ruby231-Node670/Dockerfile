FROM phusion/passenger-customizable:1.0.0
# Use phusion/passenger as base image. To make your builds reproducible, make
# sure you lock down to a specific version.
# See https://github.com/phusion/passenger-docker/blob/master/Changelog.md for
# a list of version numbers.

LABEL maintainer="Mohamed Fariz Nazeer <faariz@gmail.com>"

###############################################################################

#Set default shell to Bash
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Set root directory.
ENV HOME /root

# Install OS dependencies for compiling the application

# Install build dependencies
# Clean up APT when done.
RUN apt-get update -qq --fix-missing && \
    apt-get install -qq -y --fix-missing --no-install-recommends \
                    build-essential=12.4ubuntu1 \
                    tzdata=*-0ubuntu0.18.04 \
                    nano=2.9.* \
                    monit=1:5.25.1-1build1 && \
    apt-get autoremove -q -y && apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Set timezone for tzdata
ENV TZ=Asia/Kuala_Lumpur
RUN ln -fs /usr/share/zoneinfo/$TZ /etc/localtime && \
    echo $TZ > /etc/timezone

###############################################################################

# Install Ruby 2.3.1
RUN bash -lc 'rvm install ruby-2.3.1' && \
    bash -lc 'rvm cleanup all' && \
    bash -lc 'rvm --default use ruby-2.3.1'

###############################################################################

# Installing Node.js

# Declare Node.js Version
ENV NVM_VERSION 0.34.0
ENV NODE_VERSION 6.7.0

# Install NVM
RUN curl -o- https://raw.githubusercontent.com/creationix/nvm/v$NVM_VERSION/install.sh | bash
# Declare NVM Directory
ENV NVM_DIR $HOME/.nvm
# Install Node.js
# hadolint ignore=SC1090
RUN . $NVM_DIR/nvm.sh && \
    nvm install $NODE_VERSION && \
    nvm alias default $NODE_VERSION && \
    nvm use default
# Set Node.js binary executible
RUN ln -sf $NVM_DIR/versions/node/v$NODE_VERSION/bin/node /usr/bin/nodejs && \
    ln -sf $NVM_DIR/versions/node/v$NODE_VERSION/bin/node /usr/bin/node && \
    ln -sf $NVM_DIR/versions/node/v$NODE_VERSION/bin/npm /usr/bin/npm

###############################################################################

# Enable Nginx and Passenger
RUN rm -f /etc/service/nginx/down \
          /etc/nginx/sites-enabled/default

# Set default value for environment variables for
# RAILS_ENV, RACK_ENV, WSGI_ENV, NODE_ENV, PASSENGER_APP_ENV
# To be overwritten using PASSENGER_APP_ENV
RUN echo "passenger_app_env production;" > /etc/nginx/conf.d/00_app_env.conf

# Open Port
EXPOSE 80

# Use baseimage-docker's init process.
# Runs files in /etc/my_init.d in lexicographic order.
ENTRYPOINT ["/sbin/my_init"]
