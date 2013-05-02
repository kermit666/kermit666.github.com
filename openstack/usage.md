---
layout: page
title: "OpenStack usage"
description: ""
---
{% include JB/setup %}

Usage instructions for students
================================

Connect to the physical server

    ssh user@snowwhite.infosys.tuwien.ac.at

Authenticate

    . openrc

Show available images, flavors, keys (your key should be in there too)

    nova image-list
    nova flavor-list
    nova keypair-list

Create a new instance based on this data (so you might have to replace `"Ubuntu 12.04 Precise server"`, `m1.tiny` and `userskey`)

    nova boot --image "Ubuntu 12.04 Precise server" --flavor m1.tiny --key_name userskey testinstance

See the running instances or a single instance's details

    nova list
    nova show testinstance

Log into the instance

    nova ssh --private --login ubuntu testinstance

Great, you're in! You can do whatever you want with the machine. (*note:* ubuntu is the username you're logging in with and is the default for Ubuntu cloud images - you can later create and use a different username).

**Important:** since we don't want to run instances when it's not necessary (it occupies our cloud's resources), please suspend your instance when you don't need it

    nova suspend testinstance

This will put your vm to sleep. You can later on resume it with

    nova resume testinstance

The status of the machine is shown under `nova list`.