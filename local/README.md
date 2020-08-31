# Local virtual machine test server setup

This directory is used to configure and run a local virtual machine as a test
server. This is handy for testing out various Ansible playbooks. The goal is for
the virtual machine to be set up similar to how a barebones server is set up
by the infrastructure team.

## Pre-requisites

In addition to Ansible, this setup expects you to have [Vagrant](https://www.vagrantup.com/)
and [VirtualBox](https://www.virtualbox.org/) installed.

The first time you create the virtual machine, Vagrant will use Ansible to do
some additional configuration. The playbook that does this resides in the
provisioning directory.

The provisioning playbook will also set up an SSH key for the alfresco user so
that you can ssh without providing a password. The path to the SSH key is
specified as a var in the playbook.

## Running

Before running vagrant, if you are using a Python virtual environment for
Ansible, activate the environment. Vagrant will use Ansible to set up the box.

To start up the virtual machine, run `vagrant up`.

If you need to connect you can either use `vagrant ssh` or you can SSH as you
normally would using the virtual machine's IP address (see the Vagrantfile).

To check on the status of the machine, run `vagrant status`.

To stop the virtual machine, run `vagrant halt`.

To completely remove the virtual machine, run `vagrant destroy`.
