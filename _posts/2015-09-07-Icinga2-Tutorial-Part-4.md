---
layout: post
title: Icinga2 Tutorial Part 4 - Expanding Checks to SNMP
date: 2015-09-07 16:18:00 -05:00
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
* [Icinga2 Tutorial: Part 3 - Agent-Based Checks][4]

__EDIT (2018/12/09):__ _These guides haven't been updated since 2015. It is
possible that there are dead links, or that the configuration syntax has changed
dramatically. These posts are also some of the most popular on my blog. I plan
to do a new guide eventually, but for right now please take the following
entries with a grain of salt._

## Introduction ## {: #icinga2-part-4-introduction }
Well I have finally persuaded myself to continue writing these posts by
completely deleting all the configuration I had already set up. It is worth
noting that I have switched over to Debian Jessie, for no other reason than
to cause myself more [frustration and suffering][5]. Anyways, let’s get started.

SNMP is considered an [Agent-Based Check][6], and is actually quite
flexible. You can even go as far as to code in custom return options, to check
things you normally wouldn’t be able to check over snmp, for example,
[apt status][7], and other such things.

It is worth noting that due to using a very small LAN, I will not be
fiddling around with SNMPv3, I will be going with straight SNMPv1,
just with a modified community string. We will get started with my core
router, Djehuti. It is outside the scope of this tutorial to discuss
how to enable SNMP on your device, but if you use a Ubiquiti device,
hey that might come soon.

Starting from this post forward, I will be embedding code here instead of
referring to an external link, as embedding will encourage me to be a bit more
complete in my explanations. So, with all of that said, let’s get started.

## Initial Setup ##
To monitor SNMP we will be using the [Manubulon SNMP Plugins][8]. So we first
need to install them.

```
zyradyl@captor:~$ sudo apt-get install nagios-snmp-plugins
```

Now we need to open up the main Icinga2 Configuration file and add in the
proper include to allow us to use these plugins. You may notice while poking
around this file that there are many things you either don’t need or would like
to change. I do plan to come back to this file at a later time, but feel free to
edit this file before that happens. Once you have made the proper changes,
restart Icinga2 so the new settings take effect.

```
zyradyl@captor:~$ sudo vim /etc/icinga2/icinga2.conf

    include <manubulon>

zyradyl@captor:~$ sudo service icinga2 restart
```

With that, we can move on to creating configuration files!

## Djehuti ##
We will be starting with my core router, which is running SNMPv1. The first
thing we will want to do is to add some essential variables to our host
directive so that we don’t have to redefine them with every service.

```
//
// Host Declaration Block
//
object Host "djehuti.zyradyl.org" {
    // Define the host IPv4 Address
    address             = "10.0.0.1"
    // Define a basic functionality test
    // Hostalive does a basic ICMP ECHO to the target
    // specified in the address directive.
    check_command       = "hostalive"
    // Define SNMP Variables
    vars.snmp_address   = "10.0.0.1"
    vars.snmp_community = "zyradyl"
    // These are not strictly needed. I add them
    // so I know at a glance what version of snmp
    // I am using.
    vars.snmp_v2        = "false"
    vars.snmp_v3        = "false"
}
```

The new additions are any of the `var.snmp*` commands located under the
`check_command` line. With our host variables set up, we can now move on to
defining a service. The first service defined in the
[Icinga2 Manubulon Documentation][9] is the `snmp-load` check. Seems like a
good starting place to me!

### SNMP-Load ###

```
//    
// Service Declaration Block
// Service:     snmp_load
// Description: Uses SNMP commands to check the load averages
//              on the device.
//
object Service "snmp-load" {
    host_name           = "djehuti.zyradyl.org"
    // Set the type of load check to use.
    vars.snmp_load_type = "netsl"
    // Set the Load Average warning threshold.
    vars.snmp_warn      = "5,3,2"
    // Set the Load Average critical threshold.
    vars.snmp_crit      = "6,5,3"
    check_command       = "snmp-load"
}
```

I feel I should take a minute to explain the warning and critical variables,
because the icinga2 documentation doesn’t do a very good job. When checking
load averages on \*nix systems, there are three parameters:

 - Average Load over one minute
 - Average Load over five Minutes
 - Average Load over fifteen minutes

Since my router is a dual core device, I have set it up so that if the system
is at full load for 15 minutes, I get a warning. If it has one process over
full load for five minutes, I get a warning. If it is three processes over full
load in one minute, I want a warning. Same thing applies to critical. If you
are trying to figure out what to set your levels at, I tend to use the following
formulas:

 - **Warning:**
    - 1min: 2\*\(Number of Cores\)\+1
    - 5min: \(Number of Cores\)\+1
    - 15min: \(Number of Cores\)
 - **Critical:**
    - 1min: 3\*\(Number of Cores\)
    - 5min: 2\*\(Number of Cores\)\+1
    - 15min: \(Number of Cores\)\+1

Once you have your file saved, restart Icinga2, and check the web interface.
Your new check will likely have an _Unknown_ Status in purple, just click on
the check, and manually run it by clicking “Check Now” in the right most panel.

With that, we can move on to the next check!

### SNMP-Memory ###

```
//
// Service Declaration Block
// Service:     SNMP-Memory
// Description: Uses SNMP commands to check status of RAM
//              and swap on the device.
//
object Service "snmp-memory" {
    host_name      = "djehuti.zyradyl.org"
    // Set the Memory warning for Ram and swap Respectively.
    // Uses percents.
    vars.snmp_warn = "50,0"
    vars.snmp_crit = "80,0"
    check_command  = "snmp-memory"
}
```

The warning and critical values are expressed as percentages of the total
amount of their applicable setting. The first one applies to RAM and the second
value corresponds to swap. Restart Icinga2 and log on to the web interface to
check that the new service works.

### SNMP-Storage ###

```
//
// Service Declaration Block
// Service:     SNMP-Storage
// Description: Uses SNMP commands to check the status of disk
//              storage space.
//
object Service "snmp-storage" {
    host_name              = "djehuti.zyradyl.org"
    // Uses percents.
    vars.snmp_warn         = "50"
    vars.snmp_crit         = "80"
    // Specify which partition to monitor
    vars.snmp_storage_name = "/root.dev"
    check_command          = "snmp-storage"
}
```

The `snmp_storage_name` variable is used to specify which device you want to
check the status of. If you aren’t sure which device you need to check, set
it to blank, then let it run. It will return a list of partitions that you can
check. Simply enter the name into that variable and you are good to go.

Just as memory, `snmp-storage` uses percent values in the warning and critical
threshold variables.

### SNMP-Interfaces ###

I personally like to specify a different service block for each interface
that I am monitoring, so I am not sure if it is possible to mix interfaces
together, but I don’t see any reason it wouldn’t be possible. I’m going to
list the interface configurations below, and if any variables need to be
explained I will do that below the code.

```
//
// Service Declaration Block
// Service:     SNMP-Interface
// Description: Uses SNMP commands to check the status of
//              various network interfaces on device.
//
object Service "snmp-int-lan" {
    host_name                      = "djehuti.zyradyl.org"
    // Define interface variables.
    vars.snmp_interface            = "eth0"
    vars.snmp_interface_label      = "LAN"
    vars.snmp_interface_perf       = "true"
    vars.snmp_interface_bits_bytes = "true"
    vars.snmp_interface_megabytes  = "true"
    vars.snmp_interface_noregexp   = "true"
    vars.snmp_warncrit_percent     = "true"
    // Set warning and crits to 100 to disable.
    vars.snmp_warn                 = "100,100"
    vars.snmp_crit                 = "100,100"
    check_command                  = "snmp-interface"
}

//
// Service Declaration Block
// Service:     SNMP-Interface
// Description: Uses SNMP commands to check the status of
//              various network interfaces on device.
//
object Service "snmp-int-wan" {
    host_name                      = "djehuti.zyradyl.org"
    // Define interface variables.
    vars.snmp_interface            = "eth1"
    vars.snmp_interface_label      = "WAN"
    vars.snmp_interface_perf       = "true"
    vars.snmp_interface_bits_bytes = "true"
    vars.snmp_interface_megabytes  = "true"
    vars.snmp_interface_noregexp   = "true"
    vars.snmp_warncrit_percent     = "true"
    // Set warning and crits to 100 to disable.
    vars.snmp_warn                 = "100,100"
    vars.snmp_crit                 = "100,100"
    check_command                  = "snmp-interface"
}

//
// Service Declaration Block
// Service:     SNMP-Interface
// Description: Uses SNMP commands to check the status of
//              various network interfaces on device.
//
object Service "snmp-int-dmz" {
    host_name                      = "djehuti.zyradyl.org"
    // Define interface variables.
    vars.snmp_interface            = "eth2"
    vars.snmp_interface_label      = "DMZ"
    vars.snmp_interface_perf       = "true"
    vars.snmp_interface_bits_bytes = "true"
    vars.snmp_interface_megabytes  = "true"
    vars.snmp_interface_noregexp   = "true"
    vars.snmp_warncrit_percent     = "true"
    // Set warning and crits to 100 to disable.
    vars.snmp_warn                 = "100,100"
    vars.snmp_crit                 = "100,100"
    check_command                  = "snmp-interface"
}
```

So a few things in here need some explanation. The variable
`vars.snmp_interface` specifies which interface we will be checking.
`vars.snmp_interface_noregexp` is related to this in that it tells icinga2
to not use regex matching. `vars.snmp_interface_label` configures a label
that will be shown in the console. `vars.snmp_interface_megabytes`, and
`vars.snmp_interface_bits_bytes` tells Icinga2 that we want to see bandwidth
measured in megabits. These variables can be adjusted accordingly. Finally,
`vars.snmp_interface_perf` tells Icinga2 that we want to monitor bandwidth
usage.

As for warning and critical values, while I like to monitor my bandwidth, I
don’t actually care how high it goes, at least not at the moment. More relevant
than that is the fact that my bandwidth is much less than a gigabit, but let’s
move on from that. `vars.snmp_warncrit_percent` says that we are going to
specify our warning and critical thresholds as a percent of total available
bandwidth on that port. I then set `vars.snmp_warn`, and `vars.snmp_crit` to
100 so that it is effectively disabled.

Once activating these services, you should reset Icinga2. It is worth noting
that you will first get a pending, and then an unknown status for about five
minutes, depending on your check time. Icinga compares the newest reading to
a previous reading that is sufficently old enough, which is usually about five
minutes, to calculate what has changed. Until you have a row in the database
that is the proper age, you will get a big _Unknown_ status. Nothing to worry
about, check back in a half hour.

## Conclusion ## {: #icinga2-part-4-conclusion }
There is one more snmp check that is available, and that is the process check.
While I previously used this setup on my core router, it ended up causing some
rather wonky effects, so I have elected to not use it. This check would be
useful to monitor the status of a mission critical process, such as a webserver
or even a database server. It works by searching the process list for the number
of times a string appears, and then going from there. I may cover this in the
future, but I won’t be at the moment.

Thank you for reading, and I hope to have part five up with less of a lag time.
I am also planning to do Icinga2 integration with slack soon, so stay tuned
for that!


[1]: {% post_url 2015-08-10-Icinga2-Tutorial-Part-0 %}
[2]: {% post_url 2015-08-11-Icinga2-Tutorial-Part-1 %}
[3]: {% post_url 2015-08-12-Icinga2-Tutorial-Part-2 %}
[4]: {% post_url 2015-08-17-Icinga2-Tutorial-Part-3 %}
[5]: https://jimlynch.com/linux-articles/the-psychology-of-a-distrohopper/
[6]: https://docs.icinga.org/icinga2/latest/doc/module/icinga2/toc#!/icinga2/latest/doc/module/icinga2/chapter/agent-based-checks-addon
[7]: https://wiki.icinga.org/display/howtos/check_apt+via+SNMP
[8]: https://nagios.manubulon.com/
[9]: https://docs.icinga.org/icinga2/latest/doc/module/icinga2/toc#!/icinga2/latest/doc/module/icinga2/chapter/plugin-check-commands#snmp-manubulon-plugin-check-commands
