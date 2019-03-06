FROM phusion/passenger-customizable:1.0.0
# Use phusion/passenger as base image. To make your builds reproducible, make
# sure you lock down to a specific version.
# See https://github.com/phusion/passenger-docker/blob/master/Changelog.md for
# a list of version numbers.

LABEL maintainer="Mohamed Fariz Nazeer <fariz@kaodim.com>"

# Set root directory.
ENV HOME /root

# Use baseimage-docker's init process.
# Runs files in /etc/my_init.d in lexicographic order.
CMD ["/sbin/my_init"]

###############################################################################

# Install OS dependencies for compiling the application

# Add Passenger APT repository
# Fix gpg verification error while running apt-get uodate
RUN echo "deb [trusted=yes] https://oss-binaries.phusionpassenger.com/apt/passenger $(lsb_release -cs) main" \
    > /etc/apt/sources.list.d/passenger.list
# Add Yarn APT repository
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list

# Install build dependencies
# Yarn installation of latest stable release
RUN apt-get update && \
    apt-get install -qq -y --fix-missing --no-install-recommends \
                    build-essential \
                    tzdata \
                    yarn

# Clean up APT when done.
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

###############################################################################

# Install Ruby 2.3.1
RUN bash -lc 'rvm install ruby-2.3.1'
RUN bash -lc 'rvm --default use ruby-2.3.1'

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
RUN . $NVM_DIR/nvm.sh && \
    nvm install $NODE_VERSION && \
    nvm alias default $NODE_VERSION && \
    nvm use default
# Set Node.js binary executible
RUN ln -sf $NVM_DIR/versions/node/v$NODE_VERSION/bin/node /usr/bin/nodejs
RUN ln -sf $NVM_DIR/versions/node/v$NODE_VERSION/bin/node /usr/bin/node
RUN ln -sf $NVM_DIR/versions/node/v$NODE_VERSION/bin/npm /usr/bin/npm

###############################################################################

# Enable Nginx and Passenger
RUN rm -f /etc/service/nginx/down
RUN rm /etc/nginx/sites-enabled/default

# Set default value for environment variables for
# RAILS_ENV, RACK_ENV, WSGI_ENV, NODE_ENV, PASSENGER_APP_ENV
# To be overwritten using PASSENGER_APP_ENV
RUN echo "passenger_app_env production;" > /etc/nginx/conf.d/00_app_env.conf

# Open Port
EXPOSE 80
