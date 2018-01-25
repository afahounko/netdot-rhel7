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

## RHEL7 & CentOS7

    yum update -y
    yum groupinstall -y 'Development Tools'
    yum install -y curl vim git openssl nmap bind-utils bind expect wget bzip2 openssl-devel perl-devel perl-CPAN


## Fedora27

    dnf update -y
    dnf groupinstall -y 'Development Tools'
    dnf install -y curl vim git openssl nmap bind-utils bind expect wget bzip2 openssl-devel perl-devel perl-CPAN

Download and build dnssec-tools

    cd /usr/src
    wget https://www.dnssec-tools.org/download/dnssec-tools-2.2.tar.gz
    tar -xzf dnssec-tools-2.2.tar.gz
    ./configure
    make
    make install




