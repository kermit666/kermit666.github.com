---
layout: page
title: "Ubuntu development cheat sheet"
description: ""
---
{% include JB/setup %}

Part I. Know your packages
==========================

Searching
---------

Find a package that contains expression in its name

    aptitude search expression

or

    apt-cache search expression


Information about packages
---------------------------

Basic information (a.k.a. status) for an installed package

    dpkg -s package

To get information (version number etc.) about an uninstalled package

    apt-cache show package

Discover a package’s dependencies

    apt-cache depends package

or what other packages are dependant on this package

    apt-cache rdepends package

To find out what files are placed on what location by a certain package (that’s already installed) use

    dpkg -L package

If it's a service, you can find the files it consists of with

    dpkg-query -S service


Part II. Building packages
==========================

You can build a package’s dependencies using apt-get’s build-dep subcommand

    sudo apt-get build-dep package

Obtaining a package's source

    bzr branch ubuntu:packagename

or

    bzr branch lp:projectname

Building the package

    bzr builddeb -- -S

(`--` denotes that options to the builder follow, `-S` is to create only a source package)

or

    dpkg-buildpackage -S

(omit `-S` to build a .deb file)

Install the package

    dpkg -i ../package.deb