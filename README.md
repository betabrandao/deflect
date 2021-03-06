# Deflect

## Purpose

[Distributed Denial of
Service](https://en.wikipedia.org/wiki/Denial-of-service_attack#Distributed_attack)
(DDoS) has become a cheap and common way to shut down legitimate voices. Deflect
tries to tilt the balance back in favour of freedom of expression.

This repository contains [Ansible](https://www.ansible.com) playbooks to setup
and maintain your own DDoS mitigation infrastructure. Its implementation is
based on the [Deflect](https://deflect.ca) service run by
[eQualit.ie](https://equalit.ie), which protects hundreds of civil society web
sites and has thwarted [numerous
attacks](https://equalit.ie/deflect-labs-reporting/).

Deflect aims at making it easier to set up an infrastructure capable to absorb
large volumes of HTTP traffic without having to subscribe to expensive
commercial services or buy specialised infrastructure. It also allows you to
keep control of your data and therefore respect visitors' privacy.

## Who is it for?

Civil society organisations wanting to defend their and their partners' web
sites at reasonable cost, with trustworthy software and while safeguarding
privacy. Many organisations have emphasised the importance not to sacrifice
privacy for DDoS protection. By running your own Deflect, there will be no
third-party middleman to your web traffic.

Deflect integrates a user-friendly web-based dashboard for both users and system
administrators. It therefore makes it easier to set up a commercial,
user-oriented DDoS protection service.

## Requirements

Debian 9 (Stretch) is required on all systems involved. The package `python`
must be installed.

## Deflect infrastructure architecture

See <doc/architecture.md>.

## Software components

### Core Deflect

Core Deflect is the basic component letting you run a minimalistic but
functional DDoS protection system. It includes:

- system security hardening measures for all servers (enforcing key-based SSH
  access, for instance);
- the web caching proxy [Apache Traffic
  Server](https://trafficserver.apache.org/) (ATS) on edges, used to cache and
  serve the pages hosted on origin servers as well as detect and block malicious
  traffic thanks to our ATS module [Banjax](https://github.com/equalitie/banjax)
  and banning engine [Swabber](https://github.com/equalitie/swabber);
- the [Bind](https://www.isc.org/downloads/bind/) DNS server and our DNS-based
  load balancing engine [EdgeManage](https://github.com/equalitie/edgemanage) on
  the controller, to continuously choose the fastest edges to serve websites;
- the [Let's Encrypt](https://letsencrypt.org/) client
  [certbot](https://certbot.eff.org/) accompanied with proper system
  configuration and ATS configuraton, to be able to generate TLS certificates
  automatically for protected websites;
- a copy of the [Deflect Ansible](https://github.com/equalitie/deflect)
  repository on the controller, as some roles are used periodically to update
  edges after configuration changes on protected websites.

### Network health monitoring

This component adds health monitoring to the Deflect infrastructure, to allow
system administrators to keep an eye on the global state of the network and be
alerted in case of problem.

This monitoring will be performed by Nagios, Shinken or a similar software, to
be determined.

Key monitored components include:

- a ping of all servers involved in the infrastructure;
- the HTTP response time of all edges;
- whether or not edges configuration is up-to-date with the configuration on the
  controller;
- the validity of TLS certificates on edges for all protected websites.

### Static log analysis

This component installs software to produce static HTML pages containing
statistics graphs based on the web visits as logged by the edges. The process to
do so is:

- the log files are copied once a day from all edges to a dedicated crunching
  server;
- the crunching server processes all these logs with awstats, which produces
  HTML files and images;
- the HTML pages are sent to a dedicated machine, possibly the controller, and
  made available to be consulted via HTTP.

The crunching process can be very CPU-intensive when the amount of logs
increases, and it is recommended that a dedicated crunching machine is used as
soon as the amount of traffic becomes important. This is also the reason why the
log crunching is performed only once a day.

### Web-based dashboards

This component installs web-based dashboards, generally on a dedicated system,
or possibly on the controller. There are two dashboards:

- the user dashboard, meant to allow owners of websites protected by Deflect to
  tweak their protection parameters (caching time, TLS certificates, DNS
  records…) and look at their visit statistics from an user-friendly interface
  available in several languages;
- the administration dashboard, from which system administrators can more easily
  change settings for any website.

The dashboards make an extremely valuable component since they allow Deflect
users to control their protection configuration and they let system
administrators perform most configuration changes without the risks associated
to a `root` shell on the controller.

Concretely, changes made through the web dashboards will be pulled by the
controller and then pushed as changes onto the relevant parts: ATS configuration
files for caching-related changes, DNS zone files for DNS changes, etc.

Without dashboard, all configuration modifications must be done by editing
Ansible configuration files on the controller server, and is thus reserved to
system administrators.

Therefore, it is recommended to install the dashboard if you plan to provide a
protection for numerous third parties.

### Controller data backup

This component adds a periodic backup of the controller's key configuration and
data files to another machine. The exact backup strategy has yet to be
determined, but its main purposes are:

- be able to recover old versions of data and configuration files if necessary;
- rebuild as easily and quickly as possible a new controller should the main one
  be lost.

### Dynamic log analysis

This component allows to run interactive analysis of the web access logs
produced by ATS and the banning logs produced by Banjax. It includes:

- Elasticsearch, Logstash and Kibana (the [ELK](https://www.elastic.co/) suite),
  running on one or several dedicated machines depending on the amount of logs
  processed;
- [Filebeat](https://www.elastic.co/products/beats/filebeat), use to transmit
  the logs from the edges to the Logstash service, which itself inserts the
  events into the Elasticsearch cluster.

This replaces advantagely the static log analysis, but may need a heavier
infrastructure if the volumes are high.

## Installation

The installation procedure will be given with more details as the project
implementation comes nearer to something deployable. It will roughly be as
follows:

1. obtain some virtual and/or physical servers;
2. clone the [Deflect Ansible repository](https://github.com/equalitie/deflect);
3. configure Ansible's inventory, host and group variables;
4. use `ansible-playbook` to setup the entire infrastructure.

## Authors

The initial development of Deflect components was done by numerous
[eQualit.ie](https://equalit.ie) employees throughout the years as the service
was improving. A significant amount of this repository is based or inspired from
this work, and is also performed by eQualit.ie Deflect staff.

## Copyright and licence

Copyright 2017 Equalit.ie inc.

This software is licenced under the terms of the GNU General Public Licence,
version 2, a copy of which is in the file COPYING.
