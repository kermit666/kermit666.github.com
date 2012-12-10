---
layout: page
title: "Precise Folsom Install"
description: ""
---
{% include JB/setup %}

happy manual installation notes
===============================

These notes document my installation of OpenStack Folsom on two servers:

- controller and compute node (happy)
- compute node (doc)

Both servers have Ubuntu 12.04 x64 server "Precise pangolin" installed.

NTP
----

First we need all servers to have synced times, which is in more detail 
explained in 
[this](http://www.ubuntugeek.com/network-time-protocol-ntp-server-and-clients-setup-in-ubuntu.html) 
blog post.

On all servers:

    sudo apt-get install ntp

Edit the master's conf /etc/ntp.conf to point to some global servers 
(e.g. Ubuntu, country server, ISP server).

    server 0.at.pool.ntp.org

and add these lines for it to rely on itself when there is no internet:

    server 127.127.1.0
    fudge 127.127.1.0 stratum 10

Point the other servers to the master

    server happy.infosys.tuwien.ac.at

Restart the service on all servers:

    sudo service ntp restart

Check that they are using the master server to sync:

    sudo ntpq -np
    date


Setting software sources for Folsom
------------------------------------
As I am installing this on Ubuntu 12.04, I activated 
the [Ubuntu Cloud Archive](https://wiki.ubuntu.com/ServerTeam/CloudArchive).

    sudo apt-get install ubuntu-cloud-keyring

Append to /etc/apt/sources.list :

    # The primary updates archive that users should be using
    
    deb http://ubuntu-cloud.archive.canonical.com/ubuntu precise-updates/folsom main
    
    # Public -proposed archive mimicking the SRU process for extended testing.
    # Packages should bake here for at least 7 days. 
    #
    #deb  http://ubuntu-cloud.archive.canonical.com/ubuntu precise-proposed/folsom main


Keystone
---------
Install it

    sudo apt-get install keystone

Remove the sqlite DB

    rm /var/lib/keystone/keystone.db

Install MySQL

    sudo apt-get install python-mysqldb mysql-server

Listen to outside connections

    sudo sed -i 's/127.0.0.1/0.0.0.0/g' /etc/mysql/my.cnf
    sudo service mysql restart

Create users and tables (replace $our_pw with the actual password)

    for app in nova glance keystone; do mysql -u root --password=$our_pw -v -e "
    CREATE DATABASE ${app};
    CREATE USER ${app}dbadmin;
    GRANT ALL PRIVILEGES ON ${app}.* TO '${app}dbadmin'@'%';
    GRANT ALL PRIVILEGES ON ${app}.* TO '${app}dbadmin'@'localhost';
    SET PASSWORD FOR '${app}dbadmin'@'%'=PASSWORD('$our_pw');
    "; done

Check that you did it with

    mysql -u keystonedbadmin -p -h 128.131.172.156
    show databases;
    quit;

Edit the conf file /etc/keystone/keystone.conf to point to the DB we just created 
and set admin token to $our_pw

    connection = mysql://keystonedbadmin:$our_pw@happy.infosys.tuwien.ac.at/keystone

Restart service and sync DB

    sudo service keystone restart
    sudo keystone-manage db_sync

THIS SCRIPT DOESN'T WORK:

Create tenants, users and roles by downloading this script

    wget https://raw.github.com/openstack/keystone/master/tools/sample_data.sh

Create and source a script containing the following parameters first:

    SERVICE_ENDPOINT=happy.infosys.tuwien.ac.at
    ADMIN_PASSWORD=$our_pw
    SERVICE_PASSWORD=$our_pw

Enable the Swift and Quantum accounts by setting 
ENABLE_SWIFT and/or ENABLE_QUANTUM environment variables.

Now you can execute the sample_data.sh script.

USE THE MANUAL METHOD :(

Create an auth script keystonerc-admin containing:

    export SERVICE_TOKEN=$our_pw
    export SERVICE_ENDPOINT=http://happy.infosys.tuwien.ac.at:35357/v2.0/

chmod it to 600 and source it.

Now you can work normally like

    keystone user-list

Do the work

    keystone tenant-create --name demo --description "Default Tenant"
    keystone user-create --name admin --pass $our_pw
    keystone role-create --name admin

List all with `user-list`, `role-list` and `tenant-list`, note the ids and connect them

    keystone user-role-add --user-id 41f0850b0dfe487cad02c13c8ea45dda --role-id f5e1afbd442c4c44b014683c8bbe7ab5 --tenant-id 908a8e207bd047a49bc0717f7a4a2477

    keystone tenant-create --name service --description "Service Tenant"
    keystone user-create --tenant-id d55e96597df5435e8db2c88a10d44e82 --name glance --pass $our_pw
    keystone user-role-add --user-id 11ec90e09eb4422a8d5bcf2ebb09523b --role-id f5e1afbd442c4c44b014683c8bbe7ab5 --tenant-id d55e96597df5435e8db2c88a10d44e82
    keystone user-create --tenant-id d55e96597df5435e8db2c88a10d44e82 --name nova --pass $our_pw
    keystone user-role-add --user-id bbef35a2cc73483bbec2b5d301722230 --role-id f5e1afbd442c4c44b014683c8bbe7ab5 --tenant-id d55e96597df5435e8db2c88a10d44e82
    keystone user-create --tenant-id d55e96597df5435e8db2c88a10d44e82 --name ec2 --pass $our_pw
    keystone user-role-add --user-id eb3d5f16aebe4e54b8859e964254fab2 --role-id f5e1afbd442c4c44b014683c8bbe7ab5 --tenant-id d55e96597df5435e8db2c88a10d44e82
    keystone user-create --tenant-id d55e96597df5435e8db2c88a10d44e82 --name swift --pass $our_pw
    keystone user-role-add --user-id 8d2c2d70594443fda9431de2f4cf8c1e --role-id f5e1afbd442c4c44b014683c8bbe7ab5 --tenant-id d55e96597df5435e8db2c88a10d44e82

First part done, take a break :)

Now...

Set the keystone.conf file to have (true by default)

    [catalog]
    driver = keystone.catalog.backends.sql.Catalog

We need to create services and their endpoints

    keystone service-create \
     --name=keystone \
     --type=identity \
     --description="Keystone Identity Service"

    endpoint-create \
     --region RegionOne \
     --service-id=6316dbb78aa44b0fa8163805a45bf90f \
     --publicurl=http://happy.infosys.tuwien.ac.at:5000/v2.0 \
     --internalurl=http://happy.infosys.tuwien.ac.at:5000/v2.0 \
     --adminurl=http://happy.infosys.tuwien.ac.at:35357/v2.0

Compute service - requires a separate endpoint for each tenant

    keystone service-create \
     --name=nova \
     --type=compute \
     --description="Nova Compute Service"

    keystone endpoint-create \
     --region RegionOne \
     --service-id=59af61dd15e1449295942578fa192141 \
     --publicurl='http://happy.infosys.tuwien.ac.at:8774/v2/%(tenant_id)s' \
     --internalurl='http://happy.infosys.tuwien.ac.at:8774/v2/%(tenant_id)s' \
     --adminurl='http://happy.infosys.tuwien.ac.at:8774/v2/%(tenant_id)s'

Volume

    keystone service-create \
     --name=volume \
     --type=volume \
     --description="Nova Volume Service"

    keystone endpoint-create \
     --region RegionOne \
     --service-id=935f0e2f50764f5a9b4a31d7fd96f2f4 \
     --publicurl='http://happy.infosys.tuwien.ac.at:8776/v1/%(tenant_id)s' \
     --internalurl='http://happy.infosys.tuwien.ac.at:8776/v1/%(tenant_id)s' \
     --adminurl='http://happy.infosys.tuwien.ac.at:8776/v1/%(tenant_id)s'

Image

    keystone service-create \
     --name=glance \
     --type=image \
     --description="Glance Image Service"

    keystone endpoint-create \
     --region RegionOne \
     --service-id=c678e9ff07e54336ac80b0af45f0ba60 \
     --publicurl=http://happy.infosys.tuwien.ac.at:9292/v1 \
     --internalurl=http://happy.infosys.tuwien.ac.at:9292/v1 \
     --adminurl=http://happy.infosys.tuwien.ac.at:9292/v1

EC2

    keystone service-create \
     --name=ec2 \
     --type=ec2 \
     --description="EC2 Compatibility Layer"

    keystone endpoint-create \
     --region RegionOne \
     --service-id=c5e256189f1c4716a50142bab939c3df \
     --publicurl=http://happy.infosys.tuwien.ac.at:8773/services/Cloud \
     --internalurl=http://happy.infosys.tuwien.ac.at:8773/services/Cloud \
     --adminurl=http://happy.infosys.tuwien.ac.at:8773/services/Admin

Object storage

    keystone service-create \
     --name=swift \
     --type=object-store \
     --description="Object Storage Service"

    keystone endpoint-create \
     --region RegionOne \
     --service-id=05a8dbd1ba8c4d318d09410455d32c06 \
     --publicurl 'http://happy.infosys.tuwien.ac.at:8888/v1/AUTH_%(tenant_id)s' \
     --adminurl 'http://happy.infosys.tuwien.ac.at:8888/v1' \
     --internalurl 'http://happy.infosys.tuwien.ac.at:8888/v1/AUTH_%(tenant_id)s'

breathe...

Verify

Unset the admin variables

    unset SERVICE_TOKEN
    unset SERVICE_ENDPOINT
    echo $SERVICE_TOKEN


    curl -d '{"auth": {"tenantName": "demo", "passwordCredentials":{"username": "admin", "password": "$our_pw"}}}' -H "Content-type: application/json" http://happy:35357/v2.0/tokens | python -mjson.tool

    keystone --os-username=admin --os-password=$our_pw --os-auth-url=http://happy:35357/v2.0 token-get

Create, chmod 600 and source a client keystonerc-cli file containinhisg

    export OS_USERNAME=admin
    export OS_PASSWORD=$our_pw
    export OS_TENANT_NAME=demo
    export OS_AUTH_URL=http://128.131.172.156:5000/v2.0/

Now try `keystone token-get` etc.



Glance
---------
Install
    sudo apt-get install glance
Remove SQLite DB
    sudo rm /var/lib/glance/glance.sqlite
Edit /etc/glance/glance-api.conf
    enable_v1_api=True
    enable_v2_api=True

    admin_tenant_name = service                                                                                                 
    admin_user = glance                                                                                                      
    admin_password = $our_pw

    [paste_deploy]                                                                                
    config_file = /etc/glance/glance-api-paste.ini

    flavor = keystone

    sql_connection = mysql://glancedbadmin:$our_pw@happy.infosys.tuwien.ac.at/glance

Edit /etc/glance/glance-api-paste.ini
    [filter:authtoken]
    admin_tenant_name = service
    admin_user = glance
    admin_password = $our_pw

Restart glance-api to pick up these changed settings.

    sudo service glance-api restart

Edit /etc/glance/glance-registry.conf
    [keystone_authtoken]
    admin_tenant_name = service
    admin_user = glance
    admin_password = $our_pw

    [paste_deploy]
    config_file = /etc/glance/glance-registry-paste.ini

    flavor=keystone

    sql_connection = mysql://glancedbadmin:$our_pw@happy.infosys.tuwien.ac.at/glance


Edit /etc/glance/glance-registry-paste.ini

    [pipeline:glance-registry-keystone]
    pipeline = authtoken context registryapp

Restart glance-api to pick up these changed settings.

    service glance-api restart

Make sure the DB is not versioned

    sudo glance-manage version_control 0

Sync DB

    sudo glance-manage db_sync

Restart glance-registry and glance-api services, as root:

    sudo service glance-registry restart
    sudo service glance-api restart

Verify

    glance image-list

Create sample images

    mkdir /tmp/images
    cd /tmp/images/
    wget http://smoser.brickies.net/ubuntu/ttylinux-uec/ttylinux-uec-amd64-12.1_2.6.35-22_1.tar.gz
    tar -zxvf ttylinux-uec-amd64-12.1_2.6.35-22_1.tar.gz 

    glance image-create \
    --name="tty-linux-kernel" \
    --disk-format=aki \
    --container-format=aki < ttylinux-uec-amd64-12.1_2.6.35-22_1-vmlinuz

    glance image-create \
    --name="tty-linux-ramdisk" \
    --disk-format=ari \
    --container-format=ari < ttylinux-uec-amd64-12.1_2.6.35-22_1-loader 

On the last command be careful to input the correct ids from `glance index`

    glance image-create --name="tty-linux" --disk-format=ami --container-format=ami --property kernel_id=91a3474a-e989-4b5c-8659-47138fa91f6f --property ramdisk_id=32195e4d-52ad-4fed-a45f-f41704989d7d < ttylinux-uec-amd64-12.1_2.6.35-22_1.img 


Nova
-----

Check for virtualization support

    sudo kvm-ok

Install

    sudo apt-get install qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils

Add your user to the libvirtd group and relogin

    sudo adduser `id -un` libvirtd

Happy is an AMD-based processor so I did a

    sudo modprobe kvm
    sudo modprobe kvm-amd

and added these lines to /etc/modules

    kvm
    kvm-amd

Verify that this gives an empty list (no errors)

    virsh -c qemu:///system list

Note: Folsom offers some information about the hardware to guest machines (for better performance). `libvirt_cpu_mode=host-model` seems to be the most user-friendly (semi-automatic) option.

Preconfiguring the network...

Set promiscous mode on

    sudo ip link set eth0 promisc on

Edit /etc/network/interfaces to have a bridge interface

    # Bridge network interface for VM networks
    auto br100
    iface br100 inet static
    address 192.168.101.1
    netmask 255.255.255.0
    bridge_stp off
    bridge_fd 0

Install bridge-utils and set up br100

    sudo apt-get install bridge-utils
    sudo brctl addbr br100

Restart networking

    sudo /etc/init.d/networking restart

Installing the controller packages

    sudo apt-get install rabbitmq-server
    sudo apt-get install nova-compute nova-volume nova-novncproxy novnc nova-api nova-ajax-console-proxy nova-cert nova-consoleauth nova-doc nova-scheduler nova-network

Edit the nova.conf file according to the [docs](http://docs.openstack.org/trunk/openstack-compute/install/apt/content/compute-minimum-configuration-settings.html). I copied the one from snowwhite.

Edit the /etc/nova/api-paste.ini file, adding:

    [filter:authtoken]
    paste.filter_factory = keystone.middleware.auth_token:filter_factory
    service_protocol = http
    service_host = happy.infosys.tuwien.ac.at
    service_port = 5000
    auth_host = happy.infosys.tuwien.ac.at
    auth_port = 35357
    auth_protocol = http
    auth_uri = http://127.0.0.1:5000/
    admin_tenant_name = service
    admin_user = nova
    admin_password = $our_pw
    signing_dirname = /tmp/keystone-signing-nova

Create the nova-volumes volume group as a loopback device. See [this dodai-deploy script](https://github.com/nii-cloud/dodai-deploy/blob/master/setup-env/Ubuntu/create-volume-group.sh) for details:

    sudo apt-get install lvm2 -y
    # Specify desired size in GB
    size=60
    dd if=/dev/zero of=/home/kermit/virtual-volume.data bs=1024 count=0 seek=${size}000000
    loop_device=`sudo losetup -f`
    sudo losetup $loop_device /home/kermit/virtual-volume.data
    sudo vgcreate nova-volumes $loop_device

Persist the loopback device by creating an upstart job (e.g. /etc/init/kermit-loopback.conf):

    description     "Setup loop devices after filesystems are mounted"

    start on mounted MOUNTPOINT=/home
    task
    exec losetup /dev/loop0 /home/kermit/virtual-volume.data



Create the tables in the DB

    sudo nova-manage db sync

Start or restart all the services

    sudo start nova-api
    sudo start nova-compute
    sudo start nova-network
    sudo start nova-scheduler
    sudo start nova-novncproxy
    sudo start nova-volume
    sudo start libvirt-bin
    sudo /etc/init.d/rabbitmq-server restart 

Create network

    nova-manage network create private --multi_host=T --fixed_range_v4=192.168.100.0/24 --bridge_interface=br100 --num_networks=1 --network_size=256

This too

    sudo restart nova-cert
    sudo restart nova-consoleauth

Upon restarting services here and there you should be able to list all the services with

    nova-manage service list

Yielding

    Binary           Host                                 Zone             Status     State Updated_At
    nova-compute     happy                                nova             enabled    :-)   2012-12-05 17:30:03
    nova-scheduler   happy                                nova             enabled    :-)   2012-12-05 17:29:55
    nova-volume      happy                                nova             enabled    :-)   2012-12-05 17:29:55
    nova-network     happy                                nova             enabled    :-)   2012-12-05 17:29:57
    nova-cert        happy                                nova             enabled    :-)   2012-12-05 17:29:54
    nova-consoleauth happy                                nova             enabled    :-)   2012-12-05 17:30:02

And check that you're running Folsom compute

    sudo nova-manage version list


Sample usage
-------------

Create a security group

    nova secgroup-add-rule default tcp 22 22 0.0.0.0/0
    nova secgroup-add-rule default icmp -1 -1 0.0.0.0/0
    nova secgroup-list-rules default

Create and add a keypair

    ssh-keygen -t rsa
    nova keypair-add --pub_key ~/.ssh/id_rsa.pub mykey

Verify

    nova keypair-list
    ssh-keygen -l -f ~/.ssh/id_rsa.pub

Choose flavour and image

    nova flavor-list
    nova image-list

Boot

    nova boot --flavor m1.tiny --image tty-linux --key_name mykey --security_group default test

If there are errors connecting to the metadata server (seen in `nova console-log test`), try starting it without any keys and using the default password to log in

    nova boot --flavor m1.tiny --image tty-linux --security_group default test

The default password is:

    cubswin:)

Anyway, to ssh find the ip

    nova list

and do something like

    ssh -i ~/.ssh/id_rsa cirros@192.168.100.5


Additional compute node
------------------------

Set up ntp client.

Set software sources (cloud archive).

Install the hypervisor (if AMD, also modprobe)

    sudo apt-get install qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils

Add your user to the libvirtd group and relogin

    sudo adduser `id -un` libvirtd

Verify

    sudo kvm-ok


Set up network to be the same as on the controller...

Set promiscous mode on

    sudo ip link set eth0 promisc on

Set hosts file (our usual)

Edit /etc/network/interfaces to have a bridge interface

    # Bridge network interface for VM networks
    auto br100
    iface br100 inet static
    address 192.168.101.1
    netmask 255.255.255.0
    bridge_stp off
    bridge_fd 0

Install bridge-utils and set up br100

    sudo apt-get install bridge-utils
    sudo brctl addbr br100

Install these Nova modules

    sudo apt-get install nova-compute nova-network nova-api

Backup your old /etc/nova/nova.conf and place the controller's nova.conf there and check permissions (640 nova:nova)

(Re)start nova-api and nova-compute

    sudo restart nova-api
    sudo restart nova-compute
    sudo restart nova-network

Didn't do this on the compute node - should I have?

    nova-manage network create private --multi_host=T --fixed_range_v4=192.168.100.0/24 --bridge_interface=br100 --num_networks=1 --network_size=256

Multinode works :)

TODO
-----

2012-12-06 16:01:54 WARNING nova.common.deprecated [req-b395ec24-f6ac-493e-955a-d4fe6c501ecf None None] Deprecated Config: The root_helper option (which lets you specify a root wrapper different from nova-rootwrap, and defaults to using sudo) is now deprecated. You should use the rootwrap_config option instead.


In a Cirros image:

cloud-userdata: failed to read user data url: http://169.254.169.254/2009-04-04/user-data

Can only log in to an UEC image

https://bugs.launchpad.net/nova/+bug/1087353


the loopback device isn't set up automatically after a restart


Enabling live migrations
=========================

On the controller node:

    sudo apt-get install nfs-server

Export your instances folder for sharing by editing `/etc/exports` and adding a line

    /var/lib/nova/instances *(rw,sync,fsid=0,no_root_squash)

and restart the server

    sudo service nfs-kernel-server restart
    sudo service idmapd restart

On the compute node:

    sudo apt-get install nfs-client

Edit `/etc/fstab` to mount the shared directory by adding (and a newline)

    happy:/    /var/lib/nova/instances      nfs4    defaults    0   0

And try mounting it

    sudo mount -av

Check if both the compute node and the controller have r/w enabled for the user nova

    ls -ld /var/lib/nova/instances/ 

On both machines:

 - modify /etc/libvirt/libvirtd.conf

    listen_tls = 0
    listen_tcp = 1
    auth_tcp = "none"

 - modify /etc/init/libvirt-bin.conf

    exec /usr/sbin/libvirtd -d -l

 - modify /etc/init/libvirt-bin.conf

    env libvirtd_opts="-d -l"
 
 - modify  /etc/default/libvirt-bin

    libvirtd_opts="-d -l"

 - restart libvirt and check that it's working

    sudo stop libvirt-bin && sudo start libvirt-bin
    ps -ef | grep libvirt
 
Folsom vs. Essex changes
=========================

root_wrap changed
------------------

    2012-12-05 18:15:40 WARNING nova.common.deprecated [req-d07e6a01-1a0c-4176-8de9-75cd53d8fdca None None] Deprecated Config: The root_helper option (which lets you specify a root wrapper different from nova-rootwrap, and defaults to using sudo) is now deprecated. You should use the rootwrap_config option instead.

Solution - instead of:

    root_helper=sudo nova-rootwrap

put this option in your /etc/nova/nova.conf file:

    root_helper=sudo nova-rootwrap /etc/nova/rootwrap.conf
