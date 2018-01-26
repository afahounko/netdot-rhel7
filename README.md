# Network Documentation Tool

Netdot is an open source tool designed to help network administrators collect, organize and maintain network documentation.

Netdot was initially developed at the University of Oregon and continues to be maintained and expanded with support from volunteers.

Features include:

* Device discovery via SNMP
* Layer2 topology discovery using:
  * CDP/LLDP
  * Spanning Tree Protocol
  * Switch forwarding tables
  * Router point-to-point subnets
* IPv4 and IPv6 address space management (IPAM)
* Address space visualization
* DNS zone file generation (BIND)
* ISC DHCPD config generation
* IP and MAC address tracking
* BGP peer and Autonomous Systems tracking
* Cable plant (sites, fiber, copper, closets, circuits...)
* Contacts (departments, providers, vendors, etc.)
* Export scripts for various monitoring tools (Nagios, Sysmon, RANCID, Cacti, SmokePing)
* Multi-level user access: Admin, Operator, User

## Documentation

Read the [Netdot Manual](https://github.com/cvicente/Netdot/blob/master/doc/manual/netdot-manual.pdf)

# Requirements

Install EPEL repository

## RHEL7

    yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

>**NOTE for RHN users**

You need to also enable the 'optional' repository to use EPEL packages as they depend on packages in that repository.
This can be done by enabling the RHEL optional subchannel for RHN-Classic.
For certificate-based subscriptions see Red Hat Subscription Management Guide.
For EPEL 7, in addition to the 'optional' repository (rhel-7-server-optional-rpms), you also need to enable the 'extras' repository (rhel-7-server-extras-rpms).

## CentOS7

    yum install -y epel-release

# Step 1 - dnssec-tools

Install and compile dnssec-tools

    yum update -y
    yum groupinstall -y 'Development Tools'
    yum install -y curl vim git openssl nmap bind-utils bind expect wget bzip2 openssl-devel perl-devel perl-CPAN

Download and build dnssec-tools

    cd /usr/src
    wget https://www.dnssec-tools.org/download/dnssec-tools-2.2.tar.gz
    tar -xzf dnssec-tools-2.2.tar.gz
    cd dnssec-tools-2.2
    ./configure
    make
    make install

Finish the dnssec-tools installation

    cd /usr/src/dnssec-tools-2.2
    cp validator/etc/dnsval.conf /usr/local/etc/dnssec-tools/dnsval.conf
    cp validator/etc/resolv.conf /usr/local/etc/dnssec-tools/resolv.conf
    cp validator/etc/root.hints /usr/local/etc/dnssec-tools/root.hints

>**Tip**
The certifcate on dnssec-tools.org has expired when writing this tutorial (funny)

*   Use the following command to skip certificate checking.

    ~~~~ {.bash}
    wget --no-check-certificate https://www.dnssec-tools.org/download/dnssec-tools-2.2.tar.gz
    ~~~~



# Step 2 - Netdot

Download and install Netdot

     cd /usr/local/src/
     git clone https://github.com/cvicente/Netdot.git netdot
     cd /usr/local/src/netdot/
     make rpm-install

It will ask you some questions. Answer for `mysql` or `Pg` for the RDBMS backend and hit `ENTER` to confirm the default values for other prompts.

```
Installing required Perl modules
/usr/bin/perl bin/perldeps.pl install

Which RDBMS do you plan to use as backend: [mysql|Pg]? mysql

CPAN.pm requires configuration, but most of it can be done automatically.
If you answer 'no' below, you will enter an interactive dialog for each
configuration option instead.

Would you like to configure as much as possible automatically? [yes] ENTER

 <install_help>

Warning: You do not have write permission for Perl library directories.

To install modules, you need to configure a local Perl library directory or
escalate your privileges.  CPAN can help you by bootstrapping the local::lib
module or by configuring itself to use 'sudo' (if available).  You may also
resolve this problem manually if you need to customize your setup.

What approach do you want?  (Choose 'local::lib', 'sudo' or 'manual')
 [local::lib] ENTER

Autoconfigured everything but 'urllist'.

Now you need to choose your CPAN mirror sites.  You can let me
pick mirrors for you, you can select them from a list or you
can enter them by hand.

Would you like me to automatically choose some CPAN mirror
sites for you? (This means connecting to the Internet) [yes] ENTER
The script may ask you to create a fake password for testing purposes. You can skip that part.

[snip]

```


> **Tip**
If you are still missing Perl modules after running this step,
you can complete the process in the next step.

*   If your package manager is not supported, or if you are missing
    dependencies, you can install those by hand. However, you can at least
    take advantage of the CPAN to install Perl modules automatically.

    To test for missing modules in your system, run:

    ~~~~ {.bash}
    ~% make testdeps
    ~~~~

    Then, use this to install the missing modules:

    ~~~~ {.bash}
    ~# make installdeps
    ~~~~

    If you need to install modules individually, you can do this
    instead:

    ~~~~ {.bash}
    ~# cpan
    >install Module::Blah
    ~~~~


```

===============RESULTS===============
RRDs..............................................ok
GraphViz..........................................ok
Module::Build.....................................ok
CGI...............................................ok
Class::DBI........................................ok
Class::DBI::AbstractSearch........................ok
Apache2::Request..................................ok
HTML::Mason.......................................ok
Apache::Session...................................ok
URI::Escape.......................................ok
SQL::Translator...................................ok
SNMP::Info 2.06...................................ok
NetAddr::IP 4.042.................................ok
Apache2::AuthCookie...............................ok
Apache2::SiteControl..............................ok
Log::Dispatch.....................................ok
Log::Log4perl.....................................ok
Parallel::ForkManager.............................ok
Net::Patricia 1.20................................ok
Authen::Radius....................................ok
Test::Simple......................................ok
Test::Exception...................................ok
Net::IRR..........................................ok
Time::Local.......................................ok
File::Spec........................................ok
Net::Appliance::Session...........................ok
BIND::Config::Parser..............................ok
Net::DNS..........................................ok
Text::ParseWords..................................ok
Carp::Assert......................................ok
Digest::SHA.......................................ok
Net::DNS::ZoneFile::Fast..........................ok
Socket6...........................................ok
XML::Simple.......................................ok
DBD::mysql........................................ok

If there are still any missing Perl modules, you can try:

    make installdeps

```


# Step 3 - SNMP

Install snmp binaries

    yum install net-snmp net-snmp-utils

Download netdisco latest MIBs file

    wget http://downloads.sourceforge.net/project/netdisco/netdisco-mibs/latest-snapshot/netdisco-mibs-snapshot.tar.gz -P /tmp
    tar -zxf /tmp/netdisco-mibs-snapshot.tar.gz -C /usr/local/src
    mkdir /usr/local/netdisco
    mv /usr/local/src/netdisco-mibs-3.1 /usr/local/netdisco/mibs
    cp /usr/local/netdisco/mibs/EXTRAS/contrib/snmp.conf /etc/snmp/

Edit `/etc/snmp/snmp.conf`

    vim /etc/snmp/snmp.conf

```
# if using outside of netdisco.
mibdirs /usr/local/netdisco/mibs/rfc
mibdirs +/usr/local/netdisco/mibs/net-snmp
mibdirs +/usr/local/netdisco/mibs/cisco

# mibdirs +/usr/local/netdisco/mibs/3com
# mibdirs +/usr/local/netdisco/mibs/aerohive
# mibdirs +/usr/local/netdisco/mibs/alcatel
# mibdirs +/usr/local/netdisco/mibs/allied
# mibdirs +/usr/local/netdisco/mibs/apc

...
[snip]
...
```

>**Note**:
   Comment the unecessary mibs with `#`

Enable and start snmp servie

    systemctl enable snmpd.service
    systemctl start  snmpd.service


# Step 4 — Install Netdot

## Netdot Settings

Netdot comes with a configuration file that you need to customize to
your needs. You need to create a copy of `Default.conf` with the
name `Site.conf`

~~~~ {.bash}
cp /usr/local/src/netdot/etc/Default.conf /usr/local/src/netdot/etc/Site.conf
~~~~

Then, modify `Site.conf` to reflect your specific options. The original
file contains descriptions of each configuration item.

**Netdot will first read Default.conf and then Site.conf**

The reason for keeping two files is that when an upgrade is performed,
the `Default.conf` file can be re-written (to add new variables, etc.),
without overwriting your site-specific configuration.

> **Tip**
Notice that, each time you modify Site.conf, you must restart Apache for
the changes to take effect in the web interface.


 ~~~~ {.bash}
    systemctl restart httpd.service
 ~~~~

## Database settings

*   Install your database

    MySQL users: Install and enable MySQL server (mariadb)
    
    ~~~~ {.bash}
    yum install mariadb-server -y
    ~~~~

    Enable and start MySQL server (mariadb)
    
    ~~~~ {.bash}
    systemctl enable mariadb.service
    systemctl start  mariadb.service
    ~~~~

    Pg users: Install and enable PostgreSQL server

    ~~~~ {.bash}
    yum install postgresql-server -y
    ~~~~

    Enable and start Postgresql server
    
    ~~~~ {.bash}
    systemctl enable postgresql.service
    systemctl start  postgresql.service
    ~~~~


*   Prepare your database administrator (DBA) account

    MySQL users: The DBA account for MySQL is usually created when installing
    the package. Make sure to set a password during the installation.

    Pg users: PostgreSQL normally comes with a default DBA account named 'postgres'.
    After installing, you may need to set the password for this account as follows:

    ~~~~ {.bash}
    ~% sudo -u postgres psql postgres
    ~~~~

    Set a password for the "postgres" database role using the command:

    ~~~~
    \password postgres
    ~~~~

    and give your password when prompted. Type Control+D to exit the prompt.

*   Adjust your database configuration if necessary

    MySQL users: If you intend to use the IPAM functionalities in Netdot, you might
    need to increase the maximum packet buffer size in `my.conf` to something like:

    ~~~~
    vim /etc/my.cnf
    ~~~~

    ~~~~
    [mysqld]
    ...
    max_allowed_packet = 16M
    ...
    ~~~~

*   Adjust settings in `/usr/local/src/netdot/etc/Site.conf` with your configurations.

    ~~~~
    vim /usr/local/src/netdot/etc/Site.conf
    ~~~~

   ```
#####################################################################
# User Interface
#####################################################################

# The name of the machine or virtual host where Netdot is located
NETDOTNAME  => 'netdot.localdomain',

...

#################################################################
# Database setup
#################################################################

#
# DB_TYPE defines what sort of database NetDoT trys to talk to
# [mysql|Pg]

DB_TYPE =>  'mysql',

# DB_HOME is where the Database's commandline tools live.  $DB_HOME/bin
# should contain the binaries themselves, e.g. if "which mysql" gives
# "/usr/local/mysql/bin/mysql", $DB_HOME should be "/usr/local/mysql"

DB_HOME => '/usr',

# Set DB_DBA to the name of a DB user with permission to create new databases
# Set DB_DBA_PASSWORD to that user's password (if you don't, you'll be prompted
# later)

# For mysql, you can try 'root'.  For Pg, it is usually 'postgres'

DB_DBA          =>  'root',
DB_DBA_PASSWORD =>  '',

#
# Set this to the Fully Qualified Domain Name of your database server.
# If the database is local, rather than on a remote host, using "localhost"
# will greatly enhance performance.

DB_HOST =>  'localhost',

# If you're not running your database server on its default port,
# specify the port the database server is running on below.
# With mysql, it is usually safe to leave this blank.
# With Pg, you will probably want to use '5432'

DB_PORT => '',

#
# Set this to the canonical name of the interface NetDoT will be talking to the
# database on.  If you said that the DB_HOST above was "localhost," this
# should be too. This value will be used to grant NetDoT access to the database.
# If you want to access the NetDoT database from multiple hosts, you'll need
# to grant those database rights by hand.
#

DB_NETDOT_HOST =>  'localhost',

# set this to the name you want to give to the NetDoT database in
# your database server.

DB_DATABASE =>  'netdot',

# Set this to the name of the netdot database user

DB_NETDOT_USER =>  'netdot_user',

# Set this to the password used by the NetDoT database user
# *** Change This Before Installation***

DB_NETDOT_PASS =>  'netdot_pass',

...
   ```

## Database initialization

*   You will then be ready to initialize the database.

    ~~~~ {.bash}
    cd /usr/local/src/netdot
    make installdb 
    ~~~~

## Netdot installation

*   From the top directory in the package, do:

    ~~~~ {.bash}
    cd /usr/local/src/netdot
    make install PREFIX=/usr/local/netdot APACHEUSER=apache APACHEGROUP=apache
    ~~~~

## Apache configuration

*   Edit the supplied Apache config template for either Local, RADIUS, Kerberos or LDAP authentication, copy it to your Apache config directory and include it somewhere in your Apache configuration file:

    ~~~~ {.bash}
    cp /usr/local/netdot/etc/netdot_apache24_local.conf /etc/httpd/conf.d/
    ~~~~

*   Restart Apache:

    ~~~~ {.bash}
    systemctl restart httpd.service
    ~~~~


## CRON jobs

Netdot comes with a few scripts that should be run periodically as cron
jobs.

*   Retrieval of forwarding tables and ARP caches for IP/MAC address
    tracking

*   Devices should be re-discovered via SNMP frequently to maintain an
    accurate list of ports, ip addresses, etc.

*   Rediscovery of network topology

*   With time, old data like forwarding and ARP table entries, audit records,
    etc. should be deleted from the database to save disk space.

*   Netdot can generate text documentation that is easy to find using
    simple grepping commands, for example, information about people,
    locations, device port assignments, etc. This documentation should
    be kept up to date by exporting it frequently.

*   Configurations for external programs can be generated using Netdot
    data. See details later in this document.

*   The netdot.cron file included in the package is a sample crontab
    containing recommended periodic jobs. You should customize it to
    your liking and copy it to your cron directory, for example:

    ~~~~ {.bash}
    cp /usr/local/src/netdot/netdot.cron /etc/cron.d/netdot
    ~~~~


# Step 5 — Access Netdot

Once this is done, you can restart Apache. If you used the default
settings, point your browser to:

~~~~
http://servername.mydomain/netdot/
~~~~

You should be able to log in with:

~~~~
username: "admin"
password: "admin"
~~~~

>**Tip**
If you are using the one of the external authentication options, you should
have Netdot(radius|ldap|krb5)FailToLocal set to "yes" in your `netdot_apache2_x.conf` 
file.

>**Warning**
Please remember to change the "admin" password! Go to `Contacts ->
People`, search for 'Admin', click on [edit] and type in a new password.
Then click on the Update button.


Have fun.


# References

[netdot official documentation](https://github.com/cvicente/Netdot)

[Vadym Kalsin - DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-document-your-network-with-netdot-on-centos-7)


