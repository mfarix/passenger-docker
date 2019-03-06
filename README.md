# Docker base images for Ruby and Node.js apps running on Passenger
[![Build Status](https://travis-ci.com/mfarix/passenger-docker.svg?branch=master)](https://travis-ci.com/mfarix/passenger-docker)

Passenger-docker is a [Docker](https://www.docker.com) images meant to serve as good bases for **Ruby and Node.js** web app images.

---------------------------------------

<a name="whats_included"></a>
### What's included?

*Passenger-docker is built on top of a solid base: [baseimage-docker](http://phusion.github.io/baseimage-docker/).*

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
 * Node.js 6.7.0.
 * A build system, git, and development headers for many popular libraries, so that the most popular Ruby, Python and Node.js native extensions can be compiled without problems.

Web server and application server:

 * Nginx 1.14.
 * [Phusion Passenger 5](https://www.phusionpassenger.com/).
   * This is a fast and lightweight tool for simplifying web application integration into Nginx.
   * It adds many production-grade features, such as process monitoring, administration and status inspection.
   * It replaces (G)Unicorn, Thin, Puma, uWSGI.

<a name="memory_efficiency"></a>
### Memory efficiency

Passenger-docker is very lightweight on memory. In its default configuration, it only uses 10 MB of memory (the memory consumed by bash, runit, syslog-ng, etc).

---------------------------------------

**Relevant links:**
 [Phusion Github](https://github.com/phusion/passenger-docker)