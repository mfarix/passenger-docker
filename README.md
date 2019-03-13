# Docker base images for Ruby and Node.js apps running on Passenger
Edition|Build|Size|Version|Docker Hub
---|---|---|---|---
ruby231 | [![Build Status](https://travis-ci.com/mfarix/passenger-docker.svg?branch=master)](https://travis-ci.com/mfarix/passenger-docker) | [![](https://images.microbadger.com/badges/image/mfarix/passenger-ruby231.svg)](https://microbadger.com/images/mfarix/passenger-ruby231) | [![](https://images.microbadger.com/badges/version/mfarix/passenger-ruby231.svg)](https://microbadger.com/images/mfarix/passenger-ruby231) | [ruby231](https://hub.docker.com/r/mfarix/passenger-ruby231)
ruby231-node670 | [![Build Status](https://travis-ci.com/mfarix/passenger-docker.svg?branch=master)](https://travis-ci.com/mfarix/passenger-docker) | [![](https://images.microbadger.com/badges/image/mfarix/passenger-ruby231-node670.svg)](https://microbadger.com/images/mfarix/passenger-ruby231-node670) | [![](https://images.microbadger.com/badges/version/mfarix/passenger-ruby231-node670.svg)](https://microbadger.com/images/mfarix/passenger-ruby231-node670) | [ruby231-node670](https://hub.docker.com/r/mfarix/passenger-ruby231-node670)

Passenger-docker is a [Docker](https://www.docker.com) images meant to serve as good bases for **Ruby and Node.js** web app images.

---------------------------------------

<a name="whats_included"></a>
### What's included?

*Passenger-docker is built on top of a solid base: [baseimage-docker 0.11](http://phusion.github.io/baseimage-docker/).

Basics (learn more at [baseimage-docker](http://phusion.github.io/baseimage-docker/)):

 * Ubuntu 18.04 LTS as base system.
 * A **correct** init process ([learn more](http://blog.phusion.nl/2015/01/20/docker-and-the-pid-1-zombie-reaping-problem/)).
 * Fixes APT incompatibilities with Docker.
 * syslog-ng.
 * The cron daemon.
 * [Runit](http://smarden.org/runit/) for service supervision and management.

Language support:

 * Ruby 2.3.1
   * RVM is used to manage Ruby versions.
 * Node.js 6.7.0
   * NVM is used to manage Node.js versions.
 * A build system, git, and development headers for many popular libraries, so that the most popular Ruby, Python and Node.js native extensions can be compiled without problems.

Web server and application server:

 * Nginx 1.14.
 * [Phusion Passenger 5.3.6](https://www.phusionpassenger.com/).
   * This is a fast and lightweight tool for simplifying web application integration into Nginx.
   * It adds many production-grade features, such as process monitoring, administration and status inspection.
   * It replaces (G)Unicorn, Thin, Puma, uWSGI.

<a name="memory_efficiency"></a>
### Memory efficiency

Passenger-docker is very lightweight on memory. In its default configuration, it only uses 10 MB of memory (the memory consumed by bash, runit, syslog-ng, etc).

<a name="image_variants"></a>
### Image variants

Passenger-docker consists of several images, each one tailor made for a specific user group.

**Ruby images**

 * `mfarix/passenger-ruby231` - Ruby 2.3.1
 * `mfarix/passenger-ruby231-node670` - Ruby 2.3.1 and Nodejs 6.7.0

<a name="using"></a>
## Using the image as base

<a name="getting_started"></a>
### Getting started

Choose the one you want. See [Image variants](#image_variants).

So put the following in your Dockerfile:

    FROM mfarix/passenger-ruby231:<VERSION>
    # Or instead use one of these:
    #FROM mfarix/passenger-ruby231-node670:<VERSION>

    # ...put your own build instructions here...

<a name="app_user"></a>
### The `app` user

The image has an `app` user with UID 9999 and home directory `/home/app`. Your application is supposed to run as this user. Even though Docker itself provides some isolation from the host OS, running applications without root privileges is good security practice.

Your application should be placed inside /home/app.

Note: when copying your application, make sure to set the ownership of the application directory to `app` by calling `COPY --chown=app:app /local/path/of/your/app /home/app/webapp`

<a name="nginx_passenger"></a>
### Using Nginx and Passenger

<a name="adding_web_app"></a>
#### Adding your web app to the image

Passenger works like a `mod_ruby`, `mod_nodejs`, etc. It changes Nginx into an application server and runs your app from Nginx. So to get your web app up and running, you just have to add a virtual host entry to Nginx which describes where you app is, and Passenger will take care of the rest.

You can add a virtual host entry (`server` block) by placing a .conf file in the directory `/etc/nginx/sites-enabled`. For example:

    # /etc/nginx/sites-enabled/webapp.conf:
    server {
        listen 80;
        server_name www.webapp.com;
        root /home/app/webapp/public;

        passenger_enabled on;
        passenger_user app;
        
        passenger_ruby /usr/bin/ruby;
    }

    # Dockerfile:
    RUN rm /etc/nginx/sites-enabled/default
    ADD webapp.conf /etc/nginx/sites-enabled/webapp.conf
    RUN mkdir /home/app/webapp
    RUN ...commands to place your web app in /home/app/webapp...
    # COPY --chown=app:app /local/path/of/your/app /home/app/webapp # This copies your web app with the correct ownership.

<a name="configuring_nginx"></a>
#### Configuring Nginx

The best way to configure Nginx is by adding .conf files to `/etc/nginx/main.d` and `/etc/nginx/conf.d`. Files in `main.d` are included into the Nginx configuration's main context, while files in `conf.d` are included in the Nginx configuration's http context.

For example:

    # /etc/nginx/main.d/secret_key.conf:
    env SECRET_KEY=123456;

    # /etc/nginx/conf.d/gzip_max.conf:
    gzip_comp_level 9;

    # Dockerfile:
    ADD secret_key.conf /etc/nginx/main.d/secret_key.conf
    ADD gzip_max.conf /etc/nginx/conf.d/gzip_max.conf

<a name="nginx_env_vars"></a>
#### Setting environment variables in Nginx

By default Nginx [clears all environment variables](http://nginx.org/en/docs/ngx_core_module.html#env) (except `TZ`) for its child processes (Passenger being one of them). That's why any environment variables you set with `docker run -e`, Docker linking and `/etc/container_environment`, won't reach Nginx.

To preserve these variables, place an Nginx config file ending with `*.conf` in the directory `/etc/nginx/main.d`, in which you tell Nginx to preserve these variables. For example when linking a PostgreSQL container or MongoDB container:

    # /etc/nginx/main.d/postgres-env.conf:
    env POSTGRES_PORT_5432_TCP_ADDR;
    env POSTGRES_PORT_5432_TCP_PORT;

    # Dockerfile:
    ADD postgres-env.conf /etc/nginx/main.d/postgres-env.conf

By default, passenger-docker already contains a config file `/etc/nginx/main.d/default.conf` which preserves the `PATH` environment variable.

<a name="app_env_name"></a>
#### Application environment name (`RAILS_ENV`, `NODE_ENV`, etc)

Some web frameworks adjust their behavior according to the value some environment variables. For example, Rails respects `RAILS_ENV` while Connect.js respects `NODE_ENV`. By default, Phusion Passenger sets all of the following environment variables to the value **production**:

 * `RAILS_ENV`
 * `RACK_ENV`
 * `WSGI_ENV`
 * `NODE_ENV`
 * `PASSENGER_APP_ENV`

Setting these environment variables yourself (e.g. using `docker run -e RAILS_ENV=...`) will not have any effect, because Phusion Passenger overrides all of these environment variables. The only exception is `PASSENGER_APP_ENV` (see below).

Setting the `PASSENGER_APP_ENV` environment variable from `docker run`

    docker run -e PASSENGER_APP_ENV=staging YOUR_IMAGE

The default value is set `/etc/nginx/conf.d/00_app_env.conf`. This file will be overwritten if the user runs `docker run -e PASSENGER_APP_ENV=...`.

    # /etc/nginx/conf.d/00_app_env.conf
    passenger_app_env staging;

 <a name="running_startup_scripts"></a>
 ### Running scripts during container startup
 
 passenger-docker uses the [baseimage-docker](http://phusion.github.io/baseimage-docker/) init system, `/sbin/my_init`. This init system runs the following scripts during startup, in the following order:
 
  * All executable scripts in `/etc/my_init.d`, if this directory exists. The scripts are run during in lexicographic order.
  * The script `/etc/rc.local`, if this file exists.
 
 All scripts must exit correctly, e.g. with exit code 0. If any script exits with a non-zero exit code, the booting will fail.
 
 The following example shows how you can add a startup script. This script simply logs the time of boot to the file /tmp/boottime.txt.
 
     ### In logtime.sh (make sure this file is chmod +x):
     #!/bin/sh
     date > /tmp/boottime.txt
 
     ### In Dockerfile:
     RUN mkdir -p /etc/my_init.d
     ADD logtime.sh /etc/my_init.d/logtime.sh

<a name="inspecting_web_app_status"></a>
### Inspecting the status of your web app

If you use Passenger to deploy your web app, run:

    passenger-status
    passenger-memory-stats

<a name="logs"></a>
### Logs

If anything goes wrong, consult the log files in /var/log. The following log files are especially important:

 * /var/log/nginx/error.log
 * /var/log/syslog
 * Your app's log file in /home/app.
 
 
 Please enjoy passenger-docker.

---------------------------------------

**Relevant links:**
 [Phusion Passenger Github](https://github.com/phusion/passenger-docker)