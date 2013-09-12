---
layout: page
title: "OpenStack usage"
description: ""
---
{% include JB/setup %}

Usage instructions for students.

Creating an instance
--------------------

Connect to the physical server

    ssh user@snowwhite.infosys.tuwien.ac.at

Authenticate

    . openrc

Show available images, flavors, keys (your key should be in there too). If you need to add an image and/or key manually, check the next section.

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

Creating a keypair and an image
-------------------------------
This part might not be necessary. In case you do need to do it yourself,
here are the instructions...

### Keypair

you create and add a keypair with

    ssh-keygen -t rsa

pick a password and choose the default location.

    nova keypair-add --pub_key ~/.ssh/id_rsa.pub mykey

### Image

Download an Ubuntu server cloud image from the web - the URL of an exact image
tends to change, so try browsing to it from
[the main page](http://cloud-images.ubuntu.com/).

    wget http://uec-images.ubuntu.com/releases/12.04.2/release/ubuntu-12.04.2-server-cloudimg-amd64-disk1.img

and add it to the glance registry.

    glance image-create --is-public true --disk-format qcow2 --container-format bare --name "precise" < ubuntu-12.04.2-server-cloudimg-amd64-disk1.img

Test with

    glance index

That should be it.