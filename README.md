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

***NOTE for RHN users***

You need to also enable the 'optional' repository to use EPEL packages as they depend on packages in that repository. This can be done by enabling the RHEL optional subchannel for RHN-Classic. For certificate-based subscriptions see Red Hat Subscription Management Guide. For EPEL 7, in addition to the 'optional' repository (rhel-7-server-optional-rpms), you also need to enable the 'extras' repository (rhel-7-server-extras-rpms).


## CentOS7

    yum install -y epel-release


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

***Note***

The certifcate on dnssec-tools.org has expired when writing this tutorial :-)
Use the following command to skip certificate checking:

     wget --no-check-certificate https://www.dnssec-tools.org/download/dnssec-tools-2.2.tar.gz


# Install Netdot

## Download and build Netdot

     cd /usr/local/src/
     git clone https://github.com/cvicente/Netdot.git netdot
     cd /usr/local/src/netdot/
     make rpm-install

The prompt will ask you some questions. Answers are marked in `red`:

```
Installing required Perl modules
/usr/bin/perl bin/perldeps.pl install

Which RDBMS do you plan to use as backend: [mysql|Pg]? `mysql`

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
 [local::lib] `ENTER`

Autoconfigured everything but 'urllist'.

Now you need to choose your CPAN mirror sites.  You can let me
pick mirrors for you, you can select them from a list or you
can enter them by hand.

Would you like me to automatically choose some CPAN mirror
sites for you? (This means connecting to the Internet) [yes] `ENTER`
The script may ask you to create a fake password for testing purposes. You can skip that part.
```

The script will install the missing modules. At the end, you should see that every module installed successfully.

Run `make installdeps`  until you get all perl modules installed

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

## Install SNMP

    yum install net-snmp net-snmp-utils

### Download MIBs

    wget http://downloads.sourceforge.net/project/netdisco/netdisco-mibs/latest-snapshot/netdisco-mibs-snapshot.tar.gz -P /tmp
    tar -zxf /tmp/netdisco-mibs-snapshot.tar.gz -C /usr/local/src
    mkdir /usr/local/netdisco
    mv /usr/local/src/netdisco-mibs-3.1 /usr/local/netdisco/mibs
    cp /usr/local/netdisco/mibs/EXTRAS/contrib/snmp.conf /etc/snmp/

    vim /etc/snmp/snmp.conf

```
#        if using outside of netdisco.
mibdirs /usr/local/netdisco/mibs/rfc
mibdirs +/usr/local/netdisco/mibs/net-snmp
mibdirs +/usr/local/netdisco/mibs/cisco

mibdirs +/usr/local/netdisco/mibs/3com
#mibdirs +/usr/local/netdisco/mibs/aerohive
#mibdirs +/usr/local/netdisco/mibs/alcatel
#mibdirs +/usr/local/netdisco/mibs/allied
mibdirs +/usr/local/netdisco/mibs/apc
mibdirs +/usr/local/netdisco/mibs/arista
mibdirs +/usr/local/netdisco/mibs/aruba
mibdirs +/usr/local/netdisco/mibs/asante
mibdirs +/usr/local/netdisco/mibs/avaya
mibdirs +/usr/local/netdisco/mibs/bluecoat
mibdirs +/usr/local/netdisco/mibs/bluesocket
mibdirs +/usr/local/netdisco/mibs/cabletron
mibdirs +/usr/local/netdisco/mibs/checkpoint
mibdirs +/usr/local/netdisco/mibs/ciscosb
mibdirs +/usr/local/netdisco/mibs/citrix
mibdirs +/usr/local/netdisco/mibs/colubris
mibdirs +/usr/local/netdisco/mibs/cyclades
mibdirs +/usr/local/netdisco/mibs/d-link
mibdirs +/usr/local/netdisco/mibs/dell
mibdirs +/usr/local/netdisco/mibs/enterasys
mibdirs +/usr/local/netdisco/mibs/extreme
mibdirs +/usr/local/netdisco/mibs/extricom
mibdirs +/usr/local/netdisco/mibs/f5
mibdirs +/usr/local/netdisco/mibs/force10
mibdirs +/usr/local/netdisco/mibs/fortinet
mibdirs +/usr/local/netdisco/mibs/foundry
mibdirs +/usr/local/netdisco/mibs/gigamon
mibdirs +/usr/local/netdisco/mibs/h3c
mibdirs +/usr/local/netdisco/mibs/hp

```



