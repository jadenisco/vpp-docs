.. _settingupenvironment:

Setting up your environment
===========================

All of these exercises are designed to be performed on an Ubuntu 16.04 (Xenial) box.

* If you have an Ubuntu 16.04 box on which you have sudo or root access, you can feel free to use that.
* If you do not, a Vagrantfile is provided to setup a basic Ubuntu 16.04 box for you in the the steps below.

Install Virtual Box and Vagrant
-------------------------------

You will need to install Virtual Box and Vagrant. If you have not installed Virtual Box or Vagrant please
refer to :ref:`installingVboxVagrant` to install Virtual Box and Vagrant.

Create a Vagrant Directory
---------------------------

To get started create a directory for vagrant

.. code-block:: console

   $ mkdir vpp-tutorial
   $ cd vpp-tutorial

Create a file called **Vagrantfile** with the following contents:

.. code-block:: ruby

    # -*- mode: ruby -*-
    # vi: set ft=ruby :
    
    Vagrant.configure(2) do |config|
    
      config.vm.box = "puppetlabs/ubuntu-16.04-64-nocm"
      config.vm.box_check_update = false
    
      vmcpu=(ENV['VPP_VAGRANT_VMCPU'] || 2)
      vmram=(ENV['VPP_VAGRANT_VMRAM'] || 4096)
    
      config.ssh.forward_agent = true
    
      config.vm.provider "virtualbox" do |vb|
          vb.customize ["modifyvm", :id, "--ioapic", "on"]
          vb.memory = "#{vmram}"
          vb.cpus = "#{vmcpu}"
          #support for the SSE4.x instruction is required in some versions of VB.
          vb.customize ["setextradata", :id, "VBoxInternal/CPUM/SSE4.1", "1"]
          vb.customize ["setextradata", :id, "VBoxInternal/CPUM/SSE4.2", "1"]
      end
    end


Running Vagrant
---------------

VPP runs in userspace.  In a production environment you will often run it with
DPDK to connect to real NICs or vhost to connect to VMs.mIn those circumstances
you usually run a single instance of VPP.

For purposes of this tutorial, it is going to be extremely useful to run multiple
instances of vpp, and connect them to each other to form a topology.  Fortunately,
VPP supports this.

When running multiple VPP instances, each instance needs to have specified a 'name'
or 'prefix'.  In the example below, the 'name' or 'prefix' is "vpp1". Note that only
one instance can use the dpdk plugin, since this plugin is trying to acquire a lock
on a file.

Setting up VPP environment with Vagrant
---------------------------------------------

After setting up Vagrant, use these commands on your Vagrant directory to boot the VM:

.. code-block:: console

    $ vagrant up
    $ vagrant ssh
    $ sudo apt-get update
    $ sudo reboot -n
    $ # Wait for the VM to reboot
    $ vagrant ssh

Install VPP
------------

Now that the VM is updated, install the VPP packages.

Install VPP with the following commands:

.. code-block:: console

   $ sudo bash
   # echo "deb [trusted=yes] https://nexus.fd.io/content/repositories/fd.io.ubuntu.xenial.main/ ./" > /etc/apt/sources.list.d/99fd.io.list
   # apt-get update
   # apt-get install vpp-lib vpp vpp-plugins
   #

Stop VPP for this tutorial we will create our own instances.

.. code-block:: console

   # service vpp stop
   #

For more on installing VPP please refer to :ref:`installingVPP`.

Create some startup files
--------------------------

We will some startup files for the use of this tutorial. Typical you will
modify the startup.conf file found in /etc/vpp/startup.conf. For more information
on this file refer to :ref:`startup`.

.. code-block:: console

   $ echo "unix {cli-listen /run/vpp/cli-vpp1.sock} api-segment { prefix vpp1 } plugins { plugin dpdk_plugin.so { disable } }" > startup1.conf
   $ echo "unix {cli-listen /run/vpp/cli-vpp2.sock} api-segment { prefix vpp2 } plugins { plugin dpdk_plugin.so { disable } }" > startup2.conf
   $
