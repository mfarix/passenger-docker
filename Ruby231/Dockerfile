FROM phusion/passenger-customizable:1.0.0
# Use phusion/passenger as base image. To make your builds reproducible, make
# sure you lock down to a specific version.
# See https://github.com/phusion/passenger-docker/blob/master/Changelog.md for
# a list of version numbers.

LABEL maintainer="Mohamed Fariz Nazeer <faariz@gmail.com>"

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

# Install build dependencies
# Yarn installation of latest stable release
# Clean up APT when done.
RUN apt-get update && \
    apt-get install -qq -y --fix-missing --no-install-recommends \
                    build-essential \
                    yarn && \
    apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

###############################################################################

# Install Ruby 2.3.1
RUN bash -lc 'rvm install ruby-2.3.1' && \
    bash -lc 'rvm cleanup all' && \
    bash -lc 'rvm --default use ruby-2.3.1'

###############################################################################

# Enable Nginx and Passenger
RUN rm -f /etc/service/nginx/down && \
    rm /etc/nginx/sites-enabled/default

# Set default value for environment variables for
# RAILS_ENV, RACK_ENV, WSGI_ENV, NODE_ENV, PASSENGER_APP_ENV
# To be overwritten using PASSENGER_APP_ENV
RUN echo "passenger_app_env production;" > /etc/nginx/conf.d/00_app_env.conf

# Open Port
EXPOSE 80
