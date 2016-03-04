DevStack is a set of scripts and utilities to quickly deploy an OpenStack cloud.

# Goals

* To quickly build dev OpenStack environments in a clean Ubuntu or Fedora
  environment
* To describe working configurations of OpenStack (which code branches
  work together?  what do config files look like for those branches?)
* To make it easier for developers to dive into OpenStack so that they can
  productively contribute without having to understand every part of the
  system at once
* To make it easy to prototype cross-project features
* To provide an environment for the OpenStack CI testing on every commit
  to the projects

Read more at http://docs.openstack.org/developer/devstack

IMPORTANT: Be sure to carefully read `stack.sh` and any other scripts you
execute before you run them, as they install software and will alter your
networking configuration.  We strongly recommend that you run `stack.sh`
in a clean and disposable vm when you are first getting started.

# Versions

The DevStack master branch generally points to trunk versions of OpenStack
components.  For older, stable versions, look for branches named
stable/[release] in the DevStack repo.  For example, you can do the
following to create a juno OpenStack cloud:

    git checkout stable/juno
    ./stack.sh

You can also pick specific OpenStack project releases by setting the appropriate
`*_BRANCH` variables in the ``localrc`` section of `local.conf` (look in
`stackrc` for the default set).  Usually just before a release there will be
milestone-proposed branches that need to be tested::

    GLANCE_REPO=git://git.openstack.org/openstack/glance.git
    GLANCE_BRANCH=milestone-proposed

# Start A Dev Cloud

Installing in a dedicated disposable VM is safer than installing on your
dev machine!  Plus you can pick one of the supported Linux distros for
your VM.  To start a dev cloud run the following NOT AS ROOT (see
**DevStack Execution Environment** below for more on user accounts):

    ./stack.sh

When the script finishes executing, you should be able to access OpenStack
endpoints, like so:

* Horizon: http://myhost/
* Keystone: http://myhost:5000/v2.0/

We also provide an environment file that you can use to interact with your
cloud via CLI:

    # source openrc file to load your environment with OpenStack CLI creds
    . openrc
    # list instances
    nova list

If the EC2 API is your cup-o-tea, you can create credentials and use euca2ools:

    # source eucarc to generate EC2 credentials and set up the environment
    . eucarc
    # list instances using ec2 api
    euca-describe-instances

# DevStack Execution Environment

DevStack runs rampant over the system it runs on, installing things and
uninstalling other things.  Running this on a system you care about is a recipe
for disappointment, or worse.  Alas, we're all in the virtualization business
here, so run it in a VM.  And take advantage of the snapshot capabilities
of your hypervisor of choice to reduce testing cycle times.  You might even save
enough time to write one more feature before the next feature freeze...

``stack.sh`` needs to have root access for a lot of tasks, but uses
``sudo`` for all of those tasks.  However, it needs to be not-root for
most of its work and for all of the OpenStack services.  ``stack.sh``
specifically does not run if started as root.

DevStack will not automatically create the user, but provides a helper
script in ``tools/create-stack-user.sh``.  Run that (as root!) or just
check it out to see what DevStack's expectations are for the account
it runs under.  Many people simply use their usual login (the default
'ubuntu' login on a UEC image for example).

# Customizing

DevStack can be extensively configured via the configuration file
`local.conf`.  It is likely that you will need to provide and modify
this file if you want anything other than the most basic setup.  Start
by reading the [configuration guide](doc/source/configuration.rst) for
details of the configuration file and the many available options.

# Configure devStack with VMware
Create a new virtual network in vmware using virtual network editor 
- Created VMnet10 (unused) and configure it as below 
    - host only network adapter 
    - uncheck a host virtual adapter to this network 
    - uncheck use local DHCP service to distribute IP address to VM  
    - subnet IP: 10.10.10.0, Subnet mask: 255.255.255.0

Inside VMware virtual machine settings
- remove existing NAT NIC 
- add the first NIC and set it to custom VMnet10
- add the second NIC as NAT (default to VMnet8)
- in processors section, enable Virtualize Intel VT-x/EPT or AMD-V/RVI (this option allows nested visualization)

# Enable sudo command without entering password
- echo "<your username here> ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

# Inside ubuntu adjust network configurations 
- preferably disable network manager 
- go to /etc/network/interfaces and assign the following settings 
    - append the following to the file 
      # eth0 NAT 
      auto eth0
      iface eth0 inet static
      address 10.0.0.3
      netmask 255.255.255.0
      gateway 10.0.0.2
      broadcast 10.0.0.255
      dns-nameservers 8.8.8.8

- use ip route to check out default network route
    - If we want internet access, we should set default to the NAT 
    - ip route del default
    - route add default gw 10.0.0.2 eth0
    - ip route (verify settings)


# Enable rally plugin 
- git clone https://github.com/openstack/rally
- add enable_plugin rally https://github.com/openstack/rally master to the local.conf file

# Make local.conf file avaialble in /devstack root directory 
- I created a symbolic link of /devstack/local.conf.examples/local.conf to /devstack

# Run devstack 
- > ./stack.sh 

# Shutdown devstack: 
- > ./unstack.sh

# Remove files that Devstack installed: 
- > ./clean.sh

# Rejoin Devstack after reboot: 
- > ./rejoin-stack.sh


