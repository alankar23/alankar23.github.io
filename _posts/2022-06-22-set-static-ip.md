---
layout: post
title:  "Static ip for Linux "
author: <author_id>
tags: [linux, networking]

---

# How to set static ip for your linux machine

First, you need to make sure you have root privelges only then you can set static ip for your machine

I hope you have super user privleges so lets jump into it.

cd into etc/netplan/
    
    cd etc/netplan

Here youll find a file named something like "00-netplan". If you dont have any file create one and paste the code block below.

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
            search: []
    version: 2

Okay, now I'll explain what each line means

## Addresses 
    addresses:
    - 192.168.1.10/24

In this line `192.168.1.10` is the static ip address which I want to assign to the machine, youll need to add `/24` at the end.

## Gateway
    gateway4: 192.168.1.254

This is your network gateway, you can find this by running.
```
$ ip r | grep default
default via 192.168.1.254 dev wlp8s0 proto dhcp metric 600
```
so, `192.168.1.254` is my router ip / gateway ip.

## Nameserver

    nameservers:
        addresses:
        - 8.8.8.8
        - 8.8.4.4

This block contains your dns records, I'm using googles dns `8.8.8.8` and `8.8.4.4`. You can use cloudflairs dns or your custom dns similarly.

## Applying Changes

To apply the changes 
```
$ sudo netplan generate
```
Then,
```
$ sudo netplan apply
```

Now, reboot the system for the changes to take place.
After this you can
```
ip r
```
or 
```
hostname -I
```
youll get something like this

    
    192.168.1.10  


