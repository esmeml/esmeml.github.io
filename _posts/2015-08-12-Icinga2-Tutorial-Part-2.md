---
layout: post
title: "Icinga2 Tutorial: Part 2 - Agent-Less Checks"
date: 2015-08-12 03:00:00 -05:00
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
* [Icinga2 Tutorial: Part 3 - Agent-Based Checks][3]
* [Icinga2 Tutorial: Part 4 - Extending Checks to SNMP][4]

__EDIT (2018/12/09):__ _These guides haven't been updated since 2015. It is
possible that there are dead links, or that the configuration syntax has changed
dramatically. These posts are also some of the most popular on my blog. I plan
to do a new guide eventually, but for right now please take the following
entries with a grain of salt._

## Master Host Configuration ##
So in our last part we focused on getting your machine set up as the
[Icinga2 master controller][5]. Now we can focus on getting interoperability
setup. As always, this tutorial assumes you are sudo’d as root. You can do this
by running “sudo -s”. Then we need to set ourselves up as the master node on our
network.

```
captor zyradyl # icinga2 node wizard
Welcome to the Icinga 2 Setup Wizard!

We'll guide you through all required configuration details.

Please specify if this is a satellite setup ('n' installs a master setup) [Y/n]: n
Starting the Master setup routine…
Please specifiy the common name (CN) [captor.zyradyl.org]:
information/cli: Generating new CSR in '/etc/icinga2/pki/captor.zyradyl.org.csr'.
information/cli: Created backup file '/etc/icinga2/pki/captor.zyradyl.org.key.orig'.
information/cli: Created backup file '/etc/icinga2/pki/captor.zyradyl.org.csr.orig'.
information/base: Writing private key to '/etc/icinga2/pki/captor.zyradyl.org.key'.
information/base: Writing certificate signing request to '/etc/icinga2/pki/captor.zyradyl.org.csr'.
information/cli: Signing CSR with CA and writing certificate to '/etc/icinga2/pki/captor.zyradyl.org.crt'.
information/cli: Created backup file '/etc/icinga2/pki/captor.zyradyl.org.crt.orig'.
information/cli: Copying CA certificate to '/etc/icinga2/pki/ca.crt'.
information/cli: Created backup file '/etc/icinga2/pki/ca.crt.orig'.
information/cli: Dumping config items to file '/etc/icinga2/zones.conf'.
Please specify the API bind host/port (optional):
Bind Host []:
Bind Port []:
information/cli: Enabling the APIlistener feature.
information/cli: Updating constants.conf.
information/cli: Updating constants file '/etc/icinga2/constants.conf'.
information/cli: Updating constants file '/etc/icinga2/constants.conf'.
Done.
Now restart your Icinga 2 daemon to finish the installation!

captor zyradyl # service icinga2 restart
checking Icinga2 configuration.
[...]
captor zyradyl #
```

Note that in your case, you may see one or two warning messages, but if it
pertains to files already existing, don’t worry about it, I also clipped all
the output that [Icinga2][6] spits out when it restarts in order to save space.
These posts are long enough as it is. After restarting, if you check the
website for your master node, you will see a whole bunch of new information.
This has also established the certificates the [icinga2 protocol][7] uses for
security.

## Host Monitoring ##
Now, while it is nice to have access to the Icinga protocol, in our case we
will be working with devices that do not make the option of installing Icinga
possible. Instead, we will be monitoring through four different
protocols: SSH, SNMP(v1), HTTP/S, and ICMP.

We are going to go simple in this route through the use of
[agent-less checks.][8] Agent-less checks do not rely on having a remote
program installed, and this is useful for embedded devices that may not
have enough memory to host another program.  This is also helpful if your
client only offers one or two services and it isn’t worth taking the time to
install and configure a node setup. For example, you can check if SSH is
available on a remote host, or check if the HTTP server is alive, or even
simply see if the host is alive. We will start there.

Before we get into editing the actual files, Syntax highlighting can make life
a whole lot easier. Note, you should repeat this process with a non-privileged
user as that will give you a way to take a look at icinga2 configuration files
without having to be the root user.

```
captor conf.d # mkdir -p ~/.vim/{syntax,ftdetect}
captor conf.d # cd /usr/share/icinga2-common/syntax/
captor syntax # cp vim/syntax/icinga2.vim ~/.vim/syntax/
captor syntax # cp vim/ftdetect/icinga2.vim ~/.vim/ftdetect/
```

Now test it by opening any file under the `/etc/icinga2/conf.d/` directory. You
can find instructions to do this for nano [here][9]. Now, there are a lot of
ways to configure Icinga2. You can choose to use the pre-existing
[hosts.conf][10] and [services.conf][11], or, since the [conf.d][12] directory
is read on startup, you can use one file per host. I will be doing the latter,
as I think it keeps it a lot neater. The Access point is named Wepwawet, so
let’s get started. (Note that you clearly do not need to use my header method.
You can also indent with tabs, I mean.. if you like that kind of uncertainty in
how the file will be displayed.)

```
captor conf.d # vim /etc/icinga2/conf.d/wepwawet.conf
```

You can find the document, with syntax highlighting, [here][13].

So with this basic configuration, we get three checks:

 * An ICMP Echo (hostalive)
 * A HTTPS Up Check (http)
 * A SSH Up check (ssh)

For my HTTP check, the website uses HTTPS, and no HTTP site is available. So
by setting the SSL variable, we can ensure that just HTTPS is being checked.
This ensures we are getting an accurate reading. Now we need to ensure that our
declaration works.

```
captor conf.d # /etc/init.d/icinga2 checkconfig
checking Icinga2 configuration.
captor conf.d # /etc/init.d/icinga2 reload
checking Icinga2 configuration.
Reloading icinga2 monitoring daemon: icinga2.
captor conf.d #
```

Then log in to your web page, and you should see your new host and service
definition. If this works, then move on to the next section. If not, you
should consult the error messages printed out as well as the Icinga2 log that
can be found in `/var/log`. A useful method to check if the error is in
your configuration is to run the checks manually. Attempt to ping the
host, or ssh to the host, or access a web page. If you are unable to do
these things manually, the problem may very well be with your host.

The one final thing I would like to do is to demonstrate that you can actually
do a fairly nice amount of checks without needing to have a remote agent.
For this next example, we are going to setup a check that will verify if our
SSL certificate is still valid, or how long we have till it runs out.
Previously, this was done through an [external custom command][14] but Icinga2
now has a [TCP check plugin][15] that can do this natively.

Open your configuration file and make the edits. You can follow this file with
syntax colouring [as an example][16].

Once done, reload your configuration as demonstrated above. It is worth stating
that the reload parameter also calls checkconfig, however I prefer to run them
as two separate commands. It is also worth stating that you could combine the
SSL certificate check with the HTTPS check. I separate them because for me a
certificate expiring isn’t too big of a deal – it will only throw an error –
but the HTTPS server going down is a much bigger deal.

You can see the
[configuration of the monitoring setup for my core router here][17].

## External Resources: ##
* [Icinga2 Monitoring Remote Systems: Agentless Checks][18]
* [Icinga2 Monitoring Basics: Hosts and Services][19]
* [Icinga2 Plugin Check Commands: SSH][20]
* [Icinga2 Plugin Check Commands: SSL][21]
* [Icinga2 Monitoring Basics: Command Passing Variables][22]
* [Icinga2 Configuration Validation][23]

[1]: {% post_url 2015-08-10-Icinga2-Tutorial-Part-0 %}
[2]: {% post_url 2015-08-11-Icinga2-Tutorial-Part-1 %}
[3]: {% post_url 2015-08-17-Icinga2-Tutorial-Part-3 %}
[4]:  {% post_url 2015-09-07-Icinga2-Tutorial-Part-4 %}
[5]: https://docs.icinga.org/icinga2/latest/doc/module/icinga2/toc#!/icinga2/latest/doc/module/icinga2/chapter/distributed-monitoring#distributed-monitoring-setup-master
[6]: https://docs.icinga.org/icinga2/latest/doc/module/icinga2/toc
[7]: https://docs.icinga.org/icinga2/latest/doc/module/icinga2/toc#!/icinga2/latest/doc/module/icinga2/chapter/distributed-monitoring#distributed-monitoring-setup-satellite-client
[8]: https://docs.icinga.org/icinga2/latest/doc/module/icinga2/toc#!/icinga2/latest/doc/module/icinga2/chapter/monitoring-basics#check-commands
[9]: https://docs.icinga.org/icinga2/latest/doc/module/icinga2/chapter/getting-started#configuration-syntax-highlighting-nano
[10]: https://docs.icinga.org/icinga2/latest/doc/module/icinga2/chapter/configuring-icinga2-first-steps#hosts-conf
[11]: https://docs.icinga.org/icinga2/latest/doc/module/icinga2/chapter/configuring-icinga2-first-steps#services-conf
[12]: https://docs.icinga.org/icinga2/latest/doc/module/icinga2/chapter/configuring-icinga2-first-steps#conf-d
[13]: https://pastebin.com/twTv3bem
[14]: https://namsep.blogspot.com/2014/07/icinga-2-https-and-ssl-key-expiry-check.html
[15]: https://docs.icinga.org/icinga2/latest/doc/module/icinga2/chapter/plugin-check-commands#plugin-check-command-tcp
[16]: https://pastebin.com/H7TMnCpQ
[17]: https://pastebin.com/NnJCYuLM
[18]: https://docs.icinga.org/icinga2/latest/doc/module/icinga2/toc#!/icinga2/latest/doc/module/icinga2/chapter/monitoring-remote-systems#agent-less-checks
[19]: https://docs.icinga.org/icinga2/latest/doc/module/icinga2/toc#!/icinga2/latest/doc/module/icinga2/chapter/monitoring-basics#hosts-services
[20]: https://docs.icinga.org/icinga2/latest/doc/module/icinga2/toc#!/icinga2/latest/doc/module/icinga2/chapter/plugin-check-commands#plugin-check-command-ssh
[21]: https://docs.icinga.org/icinga2/latest/doc/module/icinga2/toc#!/icinga2/latest/doc/module/icinga2/chapter/plugin-check-commands#plugin-check-command-ssl
[22]: https://docs.icinga.org/icinga2/latest/doc/module/icinga2/toc#!/icinga2/latest/doc/module/icinga2/chapter/monitoring-basics#command-passing-parameters
[23]: https://docs.icinga.org/icinga2/latest/doc/module/icinga2/toc#!/icinga2/latest/doc/module/icinga2/chapter/cli-commands#config-validation
