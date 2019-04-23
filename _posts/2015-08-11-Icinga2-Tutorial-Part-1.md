---
layout: post
title: "Icinga2 Tutorial: Part 1 - Installation and Configuration"
date: 2015-08-11 02:00:00 -05:00
license: cc0
categories:
- Tutorials
- Networking
tags:
- Icinga2
- IcingaWeb2
---
* [Icinga2 Tutorial: Part 0 - Network Monitoring for the Masses][1]
* [Icinga2 Tutorial: Part 2 - Agent-less Checks][2]
* [Icinga2 Tutorial: Part 3 - Agent-Based Checks][3]
* [Icinga2 Tutorial: Part 4 - Extending Checks to SNMP][4]

__EDIT (2018/12/09):__ _These guides haven't been updated since 2015. It is
possible that there are dead links, or that the configuration syntax has changed
dramatically. These posts are also some of the most popular on my blog. I plan
to do a new guide eventually, but for right now please take the following
entries with a grain of salt._

## Introduction ## {: #icinga2-part-1-introduction }
I wanted to get this out fairly quickly, because I just actually did this, and
while the default [Icinga2 tutorial][5] is pretty good, it is lacking in some
areas, and since everything is fresh in my mind I wanted to go ahead and draft
this up.

It is worth noting that in this post I will assume you are root. You can get to
root as a sustained environment by using `sudo -s`.

I do not condone this for day to day usage, it is **bloody dangerous**.

So, let’s get started. Sudo to root.

```
zyradyl@captor ~ $ sudo -s
[sudo] password for zyradyl:
captor zyradyl #
```

### Installing Icinga2 ###
This assumes you are root, and on a Debian based system. For other
systems, you can find [additional documentation here][6]. Note that I follow
that documentation very closely except for a few small notes, so it should be
easy to adjust this tutorial for your needs.

#### Adding Repositories ####

```
captor zyradyl # wget -O - http://debmon.org/debmon/repo.key 2>/dev/null | apt-key add -
captor zyradyl # echo deb http://debmon.org/debmon debmon-jessie main >/etc/apt/sources.list.d/debmon.list
captor zyradyl # apt-get update
```

The first command downloads the cryptographic key for the debmon
repository and hands it over to apt. Apt then installs that key, which allows
you to verify that you are using packages signed off by and validated by that
repository. The second command is then echoing the debmon main repository into
your apt-sources, so that the software available there is added to your
system’s list, which allows you to install with just a command. The third
command tells your system to update its internal package list to incorporate
the changes. After that we can go ahead and install.

#### Installation ####

```
captor zyradyl # apt-get install icinga2 monitoring-plugins
```

This will install both [icinga2][7] and the nagios monitoring plugins that are
compatible. This gives you a very good base. Start the program through the
service command, and then set it to start on boot with `update-rc.d`.

```
captor zyradyl # service icinga2 start
captor zyradyl # update-rc.d icinga2 enable
```

Tada! You now have a data aggregator/network monitor setup! However, we
want a nice way to get information from our monitoring host, so now we need to
install the web front end.

## Installing IcingaWeb2 ##
Let’s start with all the essentials.

### Package Installation ###

```
captor zyradyl # apt-get install postgresql icinga2-ido-pgsql apache2 php5-ldap php5-imagick php5-pgsql php5-intl icingaweb2 icingaweb2-module-doc icingaweb2-module-monitoring icingaweb2-module-setup
```

This is a big one, so if you are living on about a meg down, prepare to settle
in for a bit. In order, this will install the PostgreSQL database, the
[Icinga2 database connector][8], the apache2 web server, the ldap php5 module,
the imagemagick module for php5, the postgresql php5 module, the intl php5
module, the IcingaWeb2 core, the IcingaWeb2 documentation module, the IcingaWeb2
monitoring module, and the IcingaWeb2 setup module. Note that on debian systems,
it will prompt you to allow it to setup the database. I **strongly** recommend
you say NO to that, and do it manually.

Once this completes, get everything running and added to the default runlevel.

```
captor zyradyl # service postgresql start
captor zyradyl # update-rc.d postgresql enable
captor zyradyl # service apache2 start
captor zyradyl # update-rc.d apache2 enable
```

### Database Configuration ###
Now we need to make that database. The first command defines a new user named
“*icinga*” with a password of the same name. The second command then creates a
database named **icinga**, and grants ownership to the user **icinga**.

```
captor zyradyl # cd /tmp
captor tmp # sudo -u postgres psql -c "CREATE ROLE icinga WITH LOGIN PASSWORD 'icinga';"
captor tmp # sudo -u postgres createdb -O icinga -E UTF8 icinga
```

Once this is done, edit the file `pg_hba.conf`. This file controls the way
that hosts can authenticate with your database. We want to make sure that
icinga can utilize standard password login, or md5.

```
captor tmp # vim /etc/postgresql/9.4/main/pg_hba.conf

    # icinga
    local    icinga    icinga                    md5
    host     icinga    icinga    127.0.0.1/32    md5
    host     icinga    icinga    ::1/128         md5
```

Save, and then restart the database to activate the changes. Once restarted,
we can import the [IDO SQL][9] schema into Postgres. The export line pushes
the password into the environment, thus allowing us to avoid having to type it
in.

```
captor tmp # service postgresql restart
captor tmp # export PGPASSWORD=icinga
captor tmp # psql -U icinga -d icinga < /usr/share/icinga2-ido-pgsql/schema/pgsql.sql
```

Now we need to go ahead and enable the features that icinga2 will use.

```
captor tmp # icinga2 feature enable command
captor tmp # icinga2 feature enable ido-pgsql
captor tmp # icinga2 feature enable livestatus
captor tmp # icinga2 feature enable statusdata
captor tmp # service icinga2 restart
```

The final line restarts icinga2, which is the final step to enable these
modules. Now, we need to update the configuration file for ido-pgsql to connect
it to our database that we manually set up above.

```
captor tmp # vim /etc/icinga2/features-enabled/ido-pgsql.conf
```

Fill in the needed information as appropriate. Now you need to update the
[date.timezone][10] variable in the php configuration, which can be found
below.

```
captor tmp # vim /etc/php5/apache2/php.ini
```

As an example, mine is set to “*America/Detroit*”. Now, to allow the web server
to pass commands to icinga2, we need to modify the **www-data** user into the
proper group. On Debian, this is the nagios group.

```
captor tmp # usermod -a -G nagios www-data
```

Finally, we need to use icingacli to generate our authentication token to run
initial setup.

```
captor tmp # icingacli setup token create
captor tmp # icingacli setup token show
```

The second command will show you the token should you forget it. Now open a
browser and navigate to the localhost web page. and follow along.

Something to note is that you should never select skip verification until you
have checked all your log files in `/var/log`, and you are certain everything
is correct, and the programming is just being buggy. Also keep in mind that
postgres uses **port 5432**, not the default port of 3306. Finally, you should
set a password for the postgres’ database superuser, because you are
going to need to use the superuser’s account credentials in the setup program
to create the database.

```
captor tmp # sudo -u postgres psql postgres

# \password postgres
Enter Password:
```

This will change the password, and with that, we can end this part. Next up
will be configuring hosts both capable of using the icinga2 protocol, and
configuring icinga to speak to simple hosts.

[1]:  {% post_url 2015-08-10-Icinga2-Tutorial-Part-0 %}
[2]:  {% post_url 2015-08-12-Icinga2-Tutorial-Part-2 %}
[3]:  {% post_url 2015-08-17-Icinga2-Tutorial-Part-3 %}
[4]:  {% post_url 2015-09-07-Icinga2-Tutorial-Part-4 %}
[5]:  https://docs.icinga.org/icinga2/latest/doc/module/icinga2/chapter/getting-started#setting-up-icinga2
[6]:  https://docs.icinga.org/icinga2/latest/doc/module/icinga2/chapter/getting-started#setting-up-icinga2
[7]:  https://www.icinga.org/icinga/icinga-2/
[8]:  https://docs.icinga.org/icinga2/latest/doc/module/icinga2/chapter/advanced-topics#db-ido
[9]:  https://docs.icinga.org/icinga2/latest/doc/module/icinga2/chapter/getting-started#configuring-db-ido-postgresql
[10]: https://php.net/manual/en/timezones.php
