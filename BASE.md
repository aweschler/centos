## Overview

This page provides steps to install CentOS Linux in preparation for installation and setup of repositories, web sites and other remote services.

These instructions were developed using the following ISO image.

    CentOS-6.4-x86_64-bin-DVD1.iso

This image can be retrieved from the CentOS website.

I only needed the first ISO image in the distribution because 
the hardware drivers needed for my computer are present on the first image
and I did not install any optional software that may be on the second image.

If you install to a machine that requires a driver that is stored on 
the other disk, you will be prompted to insert the disk for that image.
(untested)

These instructions minimize the amount of time you spend at the local console. 
By working from a remote console with a graphical browser, 
you can easily access these instructions and other resources during installation.

## Setup Bootable Hard Drive

Obtain the installation CD mentioned above, insert into the CD drive and reboot.

This boots to a menu with several options; do not select any of these options.
Instead, press TAB to access a boot command line.

Append the word _text_ to the command provided in the command line to
get the following.

    vmlinuz initrd=initrd.img text

Press ENTER to submit the command.
This will start a text mode install.

(I skipped the disc media test.)

Complete the subsequent screens.

When installation is complete, remove the CD from the drive and reboot.

Log in as root.

Use vi to edit _/etc/sysconfig/network-scripts/ifcfg-eth0_ (or _ifcfg-em1_).
For static IP address, set the following:

    ONBOOT=yes
    BOOTPROTO=static
    IPADDR=<ip address>
    NETMASK=<net mask>

Edit edit _/etc/sysconfig/network_ as follows.

    NETWORKING=yes
    HOSTNAME=<dns path for ip address>
    GATEWAY=<ip address of router>

Start the network service.

    service network start

Edit _/etc/resolv.conf_ to set the ip address of 2 DNS servers.
The syntax is as follows.  (Do not type the angle brackets.)

    nameserver <ip address of dns server 1>
    nameserver <ip address of dns server 2>

Test the configuration by pinging the gateway (router).

    ping <gateway ip address>

The ping command will iterate on sending ping messages to the gateway.
In my case, __I needed to wait a long time for the ping to succeed__.

## Continue from Remote Machine

Logging in from a remote machine allows you to copy and paste from these notes 
as you continue the installation.

Login remotely as root using ssh. If your remote machine is Linux, 
then connect to your server machine using the command version of ssh. 

    ssh root@<ip address of server>

What to do if ssh fails with the following error?

    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    @    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

You probably used ssh to connect to this host at the ip address before.
However, now that you are re-installing the operating system, 
this host has generated a new set of security credentials for ssh. 
Open the file _~/.ssh/known_hosts_ and delete the line 
that starts with the address of the host you are trying to connect to. 

## Update Installed Packages

Update the kernel and packages installed from CD, and then reboot.

    yum -y update
    shutdown -r now

## Set up synchronization with a time server

Set the system time from a government time server, 
and then write the new time into the hardware clock.

    yum install ntp
    ntpdate north-america.pool.ntp.org
    hwclock -w
    chkconfig --level 345 ntpd on
    service ntpd start

## Setup account for indirect root access

Install sudo to enable non-root users to become root.

    yum install sudo

To allow members of the wheel group to become root without entering a password, run visudo as follows.

    visudo

Inside visudo, uncomment the following line.

    %wheel   ALL=(ALL)   NOPASSWD: ALL

To create user turner who can become root, do the following.

    useradd -g wheel turner
    passwd turner

For extra security, you can disable remote login by root. 
To do this, add the following line to _/etc/ssh/sshd_config_.

    PermitRootLogin no

Restart sshd to make the change effective.

    service sshd restart

To operate remotely as root, you must now login as a user in the wheel group, and then use sudo to become root as follows.

    sudo su -

## Install tools (optional)

Install locate to make it easy to find files. Run updatedb if file of interest was created after last running updatedb.

    yum install mlocate
    updatedb

