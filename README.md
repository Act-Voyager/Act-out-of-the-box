Act out of the box
==================

In the final stage, as described at hacking on Act will start as simple as:

    git clone https:/github.com/Act-Voyager/Act-out-of-the-box.git
    vagrant up
    vagrant ssh

For more details, see the seven-step instructions below.

How to contribute
=================

Contributions are more than welcome at any time!

There are some issues that will show up but it seems we can work arround them,
but it would be better if those were not there in the firstplace. Please go to
https://github.com/Act-Voyager/Act-out-of-the-box/issues for any troubles and
commit your changes.

For any suggestions or issues concerning the base install of the box, visit
the repository on: https://github.com/THEMA-MEDIA/Act-out-of-the-box

About the repository
====================

This Vagrant Bootloading script installs the following:

* Act Voyager
* Apache httpd config
* A $HOME/voyager event directory

Setup / Installation
====================

0) Install VirtualBox and Vagrant, visit the following websites to download
   your specific OS installers:

   * https://www.virtualbox.org/wiki/Downloads
   * https://www.vagrantup.com/downloads.html

   Or on Ubunto, run

    $ apt-get install vagrant virtualbox

1) Clone Act-out-of-the-box Vagrant File from your favourite github account to
   your work directory on your host system. (Optionally, consider forking
   the project on github and clone that one instead.)

    $ git clone https://github.com/Act-Voyager/Act-out-of-the-box.git

3) Optionally, edit the Vagrantfile field for config.vm.network ip
   address to suit your needs. By default, the VM is set up to use DHCP
   to get an IP address. You can override this by uncommenting the
   following line in Vagrantfile:

    config.vm.network: "private_network", ip: "192.168.42.42"

5) Run "vagrant up". This downloads the required image and set up
   your Act instance.

    $ vagrant up

6) Log into the VM to start Act setup process

    $ vagrant ssh

7) Start hacking!

VagrantFile
===========

This VagrantFile mainly does the final install of Act, more specific,
Act-Voyager. 

Known Issues
============

* cpanm fails the first time, possibly due to an Unicode charater in the name
  of the Author: 'Éric Cholet'

* domain name for Apache is not qualified

* missing setup

Acknowledgements
================

Thanks to Alex Muntada, Detlev Hauschildt for install clarifications
on Act itself. The original documentation might need some polishing.

Thanks for Salve J. Nilsen for checking and editing where needed


Copyright & License
===================

(c) 2014 THEMA-MEDIA Th.J. van Hoesel

