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
