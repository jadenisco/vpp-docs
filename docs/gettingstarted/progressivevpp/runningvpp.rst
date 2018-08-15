.. _runningvpp:

Running VPP
===========

Using the files we create in :ref`settingupenvironment` we will now start and
run VPP.

VPP runs in userspace. In a production environment you will often run it
with DPDK to connect to real NICs or vhost to connect to VMs. In those
circumstances you usually run a single instance of VPP.

For purposes of this tutorial, it is going to be extremely useful to run
multiple instances of VPP, and connect them to each other to form a
topology. Fortunately, VPP supports this.

When running multiple VPP instances, each instance needs to have
specified a 'name' or 'prefix'. In the example below, the 'name' or 'prefix'
is "vpp1". Note that only one instance can use the dpdk plugin, since this
plugin is trying to acquire a lock on a file. The startup files we created
disables the dpdk plugin.

Also in our startup files notice **api-segment**. **api-segment {prefix vpp1}**
tells FD.io VPP how to name the files in /dev/shm/ for your VPP instance
differently from the default. **unix {cli-listen /run/vpp/cli-vpp1.sock}**
tells vpp to use a non-default socket file when being addressed by vppctl.

Using the files we created in setup we will start VPP.

.. code-block:: console

   $ sudo /usr/bin/vpp -c startup1.conf
   vlib_plugin_early_init:361: plugin path /usr/lib/vpp_plugins:/usr/lib64/vpp_plugins
   load_one_plugin:189: Loaded plugin: abf_plugin.so (ACL based Forwarding)
   load_one_plugin:189: Loaded plugin: acl_plugin.so (Access Control Lists)
   load_one_plugin:189: Loaded plugin: avf_plugin.so (Intel Adaptive Virtual Function (AVF) Device Plugin)
   .........
   $

If VPP does not start you can try adding **nodaemon** to the startup.conf file in the
**unix** section. This should provide more information in the output.

startup.conf example:

.. code-block:: console

   unix {nodaemon cli-listen /run/vpp/cli-vpp1.sock}
   api-segment { prefix vpp1 }
   plugins { plugin dpdk_plugin.so { disable } }

The command **vppctl** will launch a VPP shell with which you can run
VPP commands interactively.

We should now be able to execute the VPP shell and show the version.

.. code-block:: console

   $ sudo vppctl -s /run/vpp/cli-vpp1.sock
       _______    _        _   _____  ___
    __/ __/ _ \  (_)__    | | / / _ \/ _ \
    _/ _// // / / / _ \   | |/ / ___/ ___/
    /_/ /____(_)_/\___/   |___/_/  /_/
   
   vpp# show version
   vpp v18.07-release built by root on c469eba2a593 at Mon Jul 30 23:27:03 UTC 2018
   vpp#

.. note::

   Use ctrl-d to exit from the VPP shell.
