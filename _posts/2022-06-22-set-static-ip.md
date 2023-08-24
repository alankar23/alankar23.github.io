---
layout: post
title:  "Static ip for Linux "
author: <author_id>
tags: [linux, networking]

---

# Configuring a Static IP Address for Your Linux Machine

To ensure a seamless setup of a static IP for your Linux machine, it's imperative to have root privileges. This privilege level is essential for establishing a static IP effectively. Assuming you possess the necessary superuser privileges, let's delve into the process.

Navigate to the 'netplan' Directory

Commence by accessing the 'netplan' directory. This can be accomplished by executing the following command
    
    cd etc/netplan

Within the `etc/netplan` directory, a file named something like `00-netplan` awaits your attention. In case this file is absent, you should create one. Subsequently, insert the code block provided below into this file.

    # This is the network config written by Alankar
    network:
    ethernets:
        ens3:
        addresses:
        - 192.168.1.10/24
        gateway4: 192.168.1.254
        nameservers:
            addresses:
            - 8.8.8.8
            - 8.8.4.4
    version: 2

Understanding the Configuration

Let's decipher the significance of each line within the configuration:

## Static IP Address Assignment:

    addresses:
    - 192.168.1.10/24

In this segment, the address "192.168.1.10" signifies the desired static IP address for the machine. It's crucial to append "/24" at the end.

## Network Gateway Specification:
    gateway4: 192.168.1.254

This value corresponds to the network gateway or router IP. Identifying this can be achieved by executing the command:

```
$ ip r | grep default
default via 192.168.1.254 dev wlp8s0 proto dhcp metric 600
```

so, `192.168.1.254` is my router ip / gateway ip in my context..

## DNS Server Configuration:

    nameservers:
        addresses:
        - 8.8.8.8
        - 8.8.4.4

This section contains the DNS records. My preference is Google's DNS, namely `8.8.8.8` and `8.8.4.4`. Alternatively, you can opt for Cloudflare's DNS or any custom DNS of your choice.

## Applying Changes:

To implement the configured changes, execute the subsequent commands:
 
```
$ sudo netplan generate
$ sudo netplan apply
```

## Finalizing the Process:

Following the implementation of changes, a system reboot is necessary for the alterations to take effect. Once this step is concluded, you can ascertain the success of the operation by executing either of the following commands

```
ip r
```
or 
```
hostname -I
```
The output should resemble:
    
    192.168.1.10  


This confirms the successful assignment of the specified static IP to your Linux machine.