---
layout: post
title: "Icinga2 Tutorial: Part 3 - Agent-Based Checks"
date: 2015-08-17 00:00:00 -05:00
license: cc0
categories:
- Tutorials
- Networking
tags:
- Icinga2
- IcingaWeb2
---
* [Icinga2 Tutorial: Part 0 - Network Monitoring for the Masses][1]
* [Icinga2 Tutorial: Part 1 - Installation and Configuration][2]
* [Icinga2 Tutorial: Part 2 - Agent-Less Checks][3]
* [Icinga2 Tutorial: Part 4 - Extending Checks to SNMP][4]

__EDIT (2018/12/09):__ _These guides haven't been updated since 2015. It is
possible that there are dead links, or that the configuration syntax has changed
dramatically. These posts are also some of the most popular on my blog. I plan
to do a new guide eventually, but for right now please take the following
entries with a grain of salt._

For this part of the tutorial, we are actually going to be rewriting the host
file for localhost. Unfortunately, I thought the EdgeRouter would be
capable of running an [Icinga2][5] client, but it isn’t available through the
apt repositories, and after the `ntopng` incident of July and August, I am
not stressing that processor any more than it needs to be to run my network.
(It is stressed enough as it is, but that is a post for another day.)

Since I have done most of the explaining in the files, these tutorials will
start including the configuration files, and any external resources
I have used.

### Configuration File ###

```
/*
 * File:        /etc/icinga2/conf.d/captor.conf
 * Title:       Captor Host Configuration File
 * Description: Host and services definition for Icinga2 for
 *              the Captor Localhost.
 * Host:        captor.zyradyl.org
 * System:      Linux Mint Debian Edition 2
 * License:     Creative Commons Zero - Public Domain
 * Version:     1.0
 * Date:        August 16, 2015
 */

//
// Host Declaration Block
//
object Host "captor.zyradyl.org" {
    // Define the host IPv4 Address
    address         = "127.0.0.1"
    // Define the host IPv6 Address
    address6        = "::1"
    // Define the operating system
    vars.os         = "Linux"
    // Define a basic functionality test
    // Hostalive does a basic ICMP ECHO to the target
    // specified in the address directive.
    check_command   = "hostalive"
}

//
// There are two ways that we can configure Icinga2.
// The first method is to write specific files for
// every host, which is a good idea if you only have
// a few hosts, and every host has something
// different going on. The other method wold be to
// use apply service logic, which is better for
// larger deployments. I have provided links to
// both a best practices guide as well as to a guide
// on apply logic. I will be using files on a per
// host basis for my deployment.
//

// For demonstration purposes I am setting up an
// IPv6 Ping check.

//
// Service Declaration Block
// Service:     Ping6
// Description: Check if Host is responding to IPv6
// Note:        The address6 specified in the above directive  
//              is carried into the service declaration
//              blocks via specifying the host_name variable
//
object Service "ping6" {
    host_name     = "captor.zyradyl.org"
    check_command = "ping6"
}

//
// Service Declaration Block
// Service:     HTTP.server
// Description: Checks if Apache in general is up.
// Note:        The address specified in the above directive  
//              is carried into the service declaration
//              blocks via specifying the host_name variable
//
// We are specifying an additional part on the service
// because we are also going to check to make sure that
// IcingaWeb2 is running.
//
object Service "http.server" {
    host_name               = "captor.zyradyl.org"
    // This tells Icinga to check only this specific
    // page.
    vars.http_uri           = "/"
    check_command           = "http"
}

//
// Service Declaration Block
// Service:     HTTP.icingaweb2
// Description: Check if the IcingaWeb service is up.
// Note:        The address specified in the above directive  
//              is carried into the service declaration
//              blocks via specifying the host_name variable
//
object Service "http.icingaweb" {
    host_name               = "captor.zyradyl.org"
    // This tells Icinga to check only this specific
    // page.
    vars.http_uri           = "/icingaweb2"
    check_command           = "http"
}

//
// Service Declaration Block
// Service:     SSL Cert Check
// Description: Check expiration date of SSL certificates.
// Note:        The address specified in the above directive  
//              is carried into the service declaration
//              blocks via specifying the host_name variable
//
// The following is commented out because captor doesnt offer
// HTTPS.
//
//object Service "ssl" {
//    host_name                         = "wepwawet.zyradyl.org"
//    vars.ssl_port                     = "443"
//    vars.ssl_cert_valid_days_warn     = "30"
//    vars.ssl_cert_valid_days_critical = "15"
//    check_command                     = "ssl"
//}

//
// Service Declaration Block
// Service:     Disks
// Description: Checks status of all installed disks.
// Note:        The address specified in the above directive  
//              is carried into the service declaration
//              blocks via specifying the host_name variable
//
object Service "disks" {
    host_name                     = "captor.zyradyl.org"
    // We need to exclude a particular partition due to
    // access denials.
    vars.disk_partitions_excluded = "/run/user/1000/gvfs"
    check_command                 = "disk"
}

//
// Service Declaration Block
// Service:     Icinga
// Description: Checks status of Icinga Instance.
// Note:        The address specified in the above directive  
//              is carried into the service declaration
//              blocks via specifying the host_name variable
//
object Service "icinga" {
    host_name     = "captor.zyradyl.org"
    check_command = "icinga"
}

// Service Declaration Block
// Service:     Load
// Description: Checks how much load the host is under.
// Note:        The address specified in the above directive  
//              is carried into the service declaration
//              blocks via specifying the host_name variable
//
object Service "load" {
    host_name     = "captor.zyradyl.org"
    check_command = "load"
}

// Service Declaration Block
// Service:     Procs
// Description: Checks number of processes on host.
// Note:        The address specified in the above directive  
//              is carried into the service declaration
//              blocks via specifying the host_name variable
//
object Service "procs" {
    host_name     = "captor.zyradyl.org"
    check_command = "procs"
}

//
// Service Declaration Block
// Service:     SSH
// Description: Checks if SSH is available.
// Note:        The address specified in the above directive  
//              is carried into the service declaration
//              blocks via specifying the host_name variable
//
// Note:        Captor does not provide ssh services.
//              Commenting out this service.
//
//object Service "ssh" {
//    host_name     = "captor.zyradyl.org"
//    check_command = "ssh"
//}

// Service Declaration Block
// Service:     Swap
// Description: Checks status of swap space on host.
// Note:        The address specified in the above directive  
//              is carried into the service declaration
//              blocks via specifying the host_name variable
//
object Service "swap" {
    host_name     = "captor.zyradyl.org"
    check_command = "swap"
}

// Service Declaration Block
// Service:     Users
// Description: Checks number of users on the system
// Note:        The address specified in the above directive  
//              is carried into the service declaration
//              blocks via specifying the host_name variable
//
object Service "users" {
    host_name     = "captor.zyradyl.org"
    check_command = "users"
}

// Service Declaration Block
// Service:     Apt
// Description: Checks status of the apt package manager.
// Note:        The address specified in the above directive  
//              is carried into the service declaration
//              blocks via specifying the host_name variable
//
object Service "apt" {
    host_name     = "captor.zyradyl.org"
    check_command = "apt"
}

// EOF
```

### Import Notes ###
It is worth noting that this article was intended to be a quick write up to
get something online. In time, I may come back to rewrite things, but at the
very least I hope the comments in the file are helpful.

[1]: {% post_url 2015-08-10-Icinga2-Tutorial-Part-0 %} "Icinga2 Tutorial Part 0"
[2]: {% post_url 2015-08-11-Icinga2-Tutorial-Part-1 %} "Icinga2 Tutorial Part 1"
[3]: {% post_url 2015-08-12-Icinga2-Tutorial-Part-2 %} "Icinga2 Tutorial Part 2"
[4]: {% post_url 2015-09-07-Icinga2-Tutorial-Part-4 %} "Icinga2 Tutorial Part 4"
[5]: https://docs.icinga.org/icinga2/latest/doc/module/icinga2/toc "Icinga2 Official Documentation"
[6]: https://pastebin.com/KEfuNcEt "Pastebin: Captor Configuration"
