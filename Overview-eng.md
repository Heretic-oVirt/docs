# Heretic oVirt Project (HVP) - Overview

## Introduction

###  What is this about 

This project aims at the automatic (non interactive) and from-scratch (using new/recycled machines) setup of a complete enterprise infrastructure based on [oVirt][10] with [Self Hosted Engine][60] (ie with the oVirt Engine, the machine controlling the whole infrastructure, hosted as a virtual machine inside the infrastructure itself), hyperconverged (ie with [Gluster][16] storage provided by the same machines which provide virtualization services), single-fault tolerant and with advanced integrated networking (by means of [OVN][18]) and file sharing (by means of [CTDB][19]/[Samba][20]/[NFS-Ganesha][21]/[Gluster-Block][65]) functions.

By "**complete enterprise infrastructure**" we mean a solution (based on standard hardware: 64 bit Intel/AMD compatible machines, absolutely off-the-shelf) which **using free software provides all functionalities** needed by a full range of enterprises, from the smallest up to the medium/large ones (obviously by scaling the hardware specifications accordingly).

Since we realize everything, from virtualization to storage and networking, by means of software, we can say that in the end we create a "Software Defined Data Center": [SDDC][70].

###  The project

The project web site can be found at the URL:  
  
https://dangerous.ovirt.life/
  
while all the development happens on Github:  
  
https://github.com/Heretic-oVirt
  
The reason for the name "**Heretic**" (and for the tongue-in-cheek name of the web site) lays in our choice of applying some of the aforementioned technologies in a way that is not the one currently deemed "orthodox", ie "formally supported" on the "Enterprise" offerings (ie subject to payment, because bound to a support contract) of those same technologies (the "heretic" choices will be _emphasized_ below).  
  

##  The hardware side

  
Our laboratory environment contains three different setups, two physical and one virtual, as an example of what can be used to realize (or simply try out) our project.


####  Physical setup 1

* 3 x HPE Microserver G7 each one with:
    * 1 x 500 GiB SATA disk dedicated to the operating system
    * 3 x 4 TiB SATA disks dedicated to data (only 1 x 2 TiB SATA disk for data on the last server)
    * 16 GiB RAM (only 12 GiB RAM on the last server)
    * 1 x 4-port additional Gigabit network adapter
* 1 generic PC made of:
    * an HPE Pavillion about 5 years old with 6 GiB RAM and a Core2 Duo processor
    * 3 x network adapters (1 embedded and 2 on expansion Gigabit cards)
* 2 x Gigabit network switches (one Jumbo-Frame capable)
* 21 network cables (CAT-5e UTP) of proper lengths
  
#### Physical setup 2

* 3 x HPE Microserver G8 each one with:
    * 1 x 500 GiB SATA disk dedicated to the operating system
    * 3 x 4 TiB SATA disks dedicated to data (only 1 x 2 TiB SATA disk for data on the last server)
    * 16 GiB RAM (only 12 GiB RAM on the last server)
    * 1 x 4-port additional Gigabit network adapter
* 1 generic PC made of:
    * a Dell about 5 years old with 2 GiB RAM and a Core2 Duo processor
    * 5 x network adapters (1 embedded, 1 on an expansion Gigabit card and 3 on Ethernet-USB adapters)
* 1 x Gigabit network switch (Jumbo-Frame, VLAN and LACP capable)
* 26 network cables (CAT-5e UTP) of proper lengths
  
#### Virtual setup

* 3 x virtual servers each one with:
    * 1 x 64 GiB virtual disk dedicated to the operating system
    * 2 x 200 GiB virtual disks dedicated to data (only 1 x 200 GiB virtual disk for data on the last server)
    * 4 GiB RAM and 2 CPU cores (with support for nested virtualization)
    * 4 x virtual network adapters
* 1 virtual desktop machine made of:
    * 2 GiB RAM and 1 CPU core
    * 5 x virtual network adapters
* 1 virtual LAN in NAT mode and 4 isolated virtual LAN segments
  
The generic PC/VD is meant only as a support machine during setup and it is expected to be decommissioned after setup has been completed.
We also validated a mixed setup with only the generic PC/VD being virtual, hosted on a laptop with 5 network adapters (1 embedded and 4 on Ethernet-USB adapters).
Finally, a minimal setup with only one server has been recently tested (requires at least oVirt version 4.2), obviously setting apart almost all fault-tolerance functionalities (still viable as a demo / proof of concept or as an initial solution for really small deployments, capable of being scaled to three servers later on).


##  The automation technologies

  
Our solution is based:  

* for the initial part on the time-proven installation automation technology of CentOS/Fedora/RHEL: **[Kickstart][61]**
* for the following part on new configuration management technologies: **[gDeploy][62]** and **[Ansible][63]**

The above mentioned automation technologies get briefly introduced below.

####  Kickstart
  
It consists of a text file which is used to instruct the CentOS/Fedora/RHEL installation program (called Anaconda) to configure from scratch the machine in a non-interactive mode (you basically pre-fill all the answers for the questions that the interactive mode would instead ask, augmented by further choices that would not be available interactively and would instead require a huge amount of work on the hardware beforehand or on the machine itself afterwards).
The great flexibility offered by the Kickstart technology consists in the availability of optional steps which preceed and follow the step properly replacing user interaction: those optional steps are made of scripts which the Kickstart file author can elaborate at will.
  

####  Ansible


It consists of a generic configuration management solution which, starting from a textual description (several files, called playbooks, in YAML format) of the desired state for the machines (using the word "state" in the most general sense: installed packages, users and groups, active services and so on), connects to those (by means of SSH: no installed agents needed), inspects the current state (together with other parameters, called "facts") and brings the machines into the desired state.
  

####  gDeploy


It consists of a specialized Gluster/RHGS infrastructure setup automation system which, based on a textual description (in INI file format) of the desired state (IP/names of the machines member of the trusted pool, LVM disk configurations for each machine, logical volumes formatting to create the bricks, Gluster volumes creation from the bricks), automatically generates and runs the Ansible playbooks (see above) needed to reach it.
  
  
#### How we combine the automation technologies


In our specific implementation, we added into the Kickstarts (using those optional steps identified by the %pre and %post directives) some Bash scripts which, by means of standard Linux commands available inside the minimal installation environment and optionally recognizing some custom parameters (to change our default settings) passed by the boot kernel commandline (or by configuration fragments retrieved together with the Kickstart file), perform an automated discovery and configuration of the whole hardware setup: network connections (find which port is connected to which network), storage resources assignments (how to use each one of the available disks), single machine role (define the identity of the node inside the oVirt machines cluster), software parameters settings (conforming, whenever possible, to known "best practices") and so on.

Those configurations do not only affect the installation part, but also create from scratch full configuration files for further functionalities which are usually not made available by interactive installations, eg a self-sufficient network resolution system (DNS) both for local system names (those of the machines we are going to configure/create) and for Internet names.
  
In fact, we aim at use cases not only inside full-blown settings; in other terms, we do not assume to be deployed inside a fully formed enterpise framework with already present basic facilities, consequently we added to our solution all the functionalities which allow a "from scratch" approach for the enterprise network environment: what we listed above in our sample setups is all that is needed, nothing more is "hidden/assumed" (except for an Internet connection).

At the end of the Kickstart-automated installations of the support PC/VD and of the servers, we use gDeploy and Ansible to automatically orchestrate all the further configurations which will set up the storage, virtualization, networking and all the specific virtual machines offering other services to the enterprise network.


##  The actual procedure
  

Let's now detail the actual procedure.  
  

####  The support PC/VD
  
Start from connecting the suppport PC/VD:  
  
* on one network port (usually the "main", embedded one) the PC/VD must be connected to the Internet (eg by means of a router / access point)
* on the other network ports the PC/VD must be connected to the available network switches

##### The networks
  
The network switches mentioned above must create **from one to four distinct networks** (ie isolated from each other and from the rest of the network environment, Internet included, either by being realized on simple unconnected distinct switches or by being realized as different VLANs on more capable switches):

1. If the PC/VD has only one further network port (beyond the one connected to the Internet), this must be connected to a separate network: the **management** one (this network will allow the communication between the servers, as oVirt nodes, and the oVirt Engine).
2. If the PC/VD has one additional further network port (like on our aforementioned example setups), this must be connected to a further separate network: the **Gluster** network (this network will allow the synchronization communications between the servers, as Gluster nodes).
3. If the PC/VD has another one additional further network port, this must be connected to another further separate network: the **production** network (ie the network to which also users' client PCs and any other pre-existing server/appliance are connected and consequently also the network to which our future virtual machines and Samba/Gluster-NFS file sharing services will "accede").
4. Finally, if the PC/VD has one additional further network port, this must be connected to a further separate network: the **isolation** network (this network allows to separate the production network from the Internet and/or from other networks).
  
During the subsequent installation phase, the Kickstart will recognize network ports actually connected and will assign to the logical networks following the order listed above (further control on those networks, from IP addressing to MTU, can be exercised by means of kernel commandline parameters or by custom configuration files).
  
Obviously, if less than 4 separate networks are available, then the communications related to the missing separate networks will conflate on the actually present ones (we must stress the fact that it is mostly "difficult" the coexistence of oVirt management and Gluster synchronization traffic on the same network, and this is why we always have at least two separate networks in our setups: in this case the second network gets automatically relegated to the almost exclusive use of Gluster synchronization while the other network takes hold of all other functions).

##### Fault tolerance

A few words on fault tolerance: to have a fault-tolerant solution (and we always mean tolerant to a single fault at a time) all base components must be redundant; we have at least 3 servers (3 and not 2 to always have a qualified majority, ie a "quorum") in our infrastructure, but obviously the use of single switches (eg one single switch for each separated network or a single VLAN-capable switch for all separated networks) would represent a single weak point (SPOF: "single point of failure") which would be enough to invalidate the fault-tolerance of the whole infrastructure; our example setups do not include that for simplicity, but all switches should always be double and stacked (2 stacked switches for each separated network or 2 stacked VLAN-capable switches for all separated networks) and all server network ports should always be connected to separated networks at least in pairs.  

##### The support PC/VD installation

Once the installation of the support PC/VD (this too managed by a dedicated Kickstart) has been started (eg from a standard CentOS7 DVD), after having optionally specified on the kernel commandline (apart from the location of the aforementioned Kickstart, for example with inst.ks=https://dangerous.ovirt.life/hvp-repos/el7/ks/heresiarch.ks) any custom parameters (all prefixed with hvp_; the complete list with explanation and default values is provided in the comments at the top of each Kickstart file), you need only to wait for the automatic restart to find yourself in front of the classic CentOS7 GNOME3 graphical login (username and password to login are customizable and default values are documented in the internal Kickstart comments).
  
At the end of the installation, the support PC/VD will be immediately ready to provide all the needed services to the aforementioned separated networks:

* DNS (for name resolution on the separated network and on the Internet)
* NTP (for time reference)
* DHCP and PXE on the management network (for network boot during the following installations)
* A local mirror (partial) of the HTTP site of the project (to provide the files needed for the following installations)
* A gateway towards the general Internet and a transparent HTTP proxy (to speed up package installations from the Internet)
* A repository for Bash scripts and Ansible playbooks to automate the final phase of the configuration process (after all servers have been installed)
  
  
####  The servers

The next step (to be repeated for each of the server machines that will make up the permanent infrastructure) is to connect the servers to the separate networks.

##### The servers network connections

Depending on the availability of network ports on the servers (even a single port per separate network is supported, obviously losing the redundancy in that case), it is possible to connect (always arbitrarily) more than one network port of each server to the same separate network (ie dedicated switch or VLAN) and, in the subsequent installation phase, the Kickstart (unique for all servers) will recognize and interpret this situation as an intention to unify the links for redundancy and load balancing (the so-called "bonding", with bonding mode defined by default parameters and controllable by means of kernel commandline options or configuration fragments).

The list of supported separate networks is the same as listed above in the case of the support PC/VD (but in the servers case the order of the networks no longer matters, because the presence of the support PC/VD allows the Kickstart to identify exactly which network each port is connected to) and, in the subsequent installation phase, the Kickstart will recognize the network ports arbitrarily connected and will assign them to the various networks (further control on those networks, from IP addressing to MTU, can be exercised by means of kernel commandline parameters or by custom configuration files).

##### The remote management controllers

Any server remote management hardware (called iLO for HPE servers like ours or iDRAC for Dell servers, but other vendors have similar, sometimes optional, solutions under the name of BMC or IPMI) should be connected to the separate management network and the support PC/VD can be used to access the remote console offered by these out-of-band management solutions.

##### The servers installation

The actual installation of the server machines is expected to happen by means of network boot (PXE), which means that on the server machines the network boot option must be activated by firmware (BIOS or UEFI) and there must be at least one of the network ports selected as a boot device (an option which is again dependent on the firmware, but the first of the embedded network cards is usually a safe choice to make) and connected to the management network mentioned above (that is: connected to the appropriate dedicated switch or VLAN).

Once the server machine has been booted from the network, it will present a boot menu with precompiled entries (it will have been created by the installation of the support PC/VD, also propagating any changes to the defaults made through kernel commandline options, changes that should not be repeated) and the only interactive choice to be made will be to select the type (a complete server based on CentOS, referred to as "Host", or a minimal server based on oVirt-NextGenerationNode, referred to as "NGN") and identity (identified as "Node 0", "Node 1" or "Node 2") of the machine you are installing, then press Enter and wait for the prompt to signal that installation has been completed.

Behind the scenes, the support PC/VD will instruct the servers to install themselves through Kickstart with a kernel command line containing the above cited custom parameters (apart from the aforementioned Kickstart directive, for example inst.ks=https://dangerous.ovirt.life/hvp-repos/el7/ks/heretic-ngn.ks which will allow the Kickstart logic to automatically detect all pre-created configuration fragments).

We must stress the fact that the server Kickstart contains a custom logic (always controllable via custom options from the kernel commandline or configuration fragments) to choose on which disk to install the operating system; by default it will be the first among those with the smallest available size: it will then be the task of the user to make sure that this disk is actually bootable, acting on the firmware (BIOS or UEFI) of the server machine.

  

####  The final configuration

  
Once the three server machines have been installed, you can proceed with the automated configuration of the actual functions.
At any "fixed point" reached by the automation it is always possible to inspect the status and even proceed from that point onwards in manual/interactive mode (for example at this point in case the "NGN" type has been chosen for the servers, you could access the [Cockpit][24] web interface of the "Node 0" from the support PC/VD and then continue interactively from there as per oVirt documentation): our aim is to automate but always maintain full compatibility and traceability of the steps, ie do not create a "closed" system.

The automated configuration (partly under development) that follows the installation step is based on Ansible playbooks (created by the installation on the support PC/VD under /usr/local/etc/hvp-ansible as well as in /etc/ansible/hosts and under /etc/ansible/group_vars) which perform (in addition to service steps such as the propagation of SSH keys and the retrieval of data on nodes, such as the number and size of available disks, to drive the subsequent logic) the following ordered steps:  

1. the gDeploy configuration file gets generated and used to create the shared storage layer based on Gluster (a corresponding interactive step is available through the Node Cockpit web interface) with an automated logic of disk choice in order to compose each one of the expected Gluster volumes (we support up to three disks on each server in order to create Gluster volumes dedicated to: oVirt Engine, other oVirt vms, ISO images, CTDB/NFS-Ganesha locking/clustering, Gluster-Block for iSCSI services, CIFS/Windows file sharing and NFS/Unix file sharing)
2. the oVirt Self Hosted Engine answers file gets generated and the installation performed on "Node 0" (a corresponding interactive step is available through the Node Cockpit web interface)
3. the Gluster storage domains get configured in oVirt (importing the Datacenter main storage domain, an action which causes the automatic addition of the Self Hosted Engine storage domain and subsequently of the Engine vm within)
4. further nodes beyond "Node 0" gets added to the oVirt cluster 
5. OVN and further (beyond management) networks get configured both on the Engine and on nodes (creating a couple of further internal isolated OVN networks, for security/test purposes)
6. CTDB, Samba, Gluster-NFS and Gluster-block services gets configured and started (Gluster-NFS will be replaced by NFS-Ganesha in the not-too-distant future), all of those relying on Gluster volumes (here lies the first _heresy_: we use the hyperconverged storage also for file sharing, not only for virtualization)
7. NFS storage domains get configured in oVirt (currently only the ISO domain)
8. additional virtual machines get created on the oVirt infrastructure (installed from-scratch by means of dedicated Kickstarts; this automation is still being developed):
    1. an Active Directory domain controller with automated from-scratch creation of a new domain (another _heresy_: we use CentOS7 with a modified Fedora Samba package rebuilt to activate DC functions using internal Heimdal Kerberos libraries etc.)
    2. a printer server member of the Active Directory domain above
    3. a database server member of the Active Directory domain above (CentOS7 with a choice of: PostgreSQL, MySQL, Firebird, MongoDB or, real _heresy_ ;-) , SQLServer!)
    4. a CentOS7 remote desktop server ([X2Go][68] based) member of the Active Directory domain above
    5. other optional vms (still being determined/developed: an ERP server using the DB server above, a messaging server, a firewall/proxy/VPN server ecc. all being members of the Active Directory domain above)
9. Samba as a file server on the Gluster nodes gets reconfigured as an Active Directory (see above) domain member
10. A complete backup infrastructure ([Bareos][27] based) gets installed and configured

The starting (as root user from the support PC/VD) of the steps above must be performed manually (eg by issuing "ansible-playbook /usr/local/etc/hvp-ansible/site.yaml"), but this is only meant to give way to the person performing the setup to choose the appropriate time and maybe to apply manual tweaks to parameters and steps beforehand: after the launching of the playbook, everything happens non-interactively.

All the software used has been made available (if modified in the course of the project) in custom [yum repositories][25] together with the corresponding source packages (all packages have been produced using mock and have beeen signed with the [project GPG key][26]).

We must stress tha fact that all the tests and the development performed so far have been mainly concerned with a set of **exactly 3** **server machines**, and in this case one of those 3, to be installed as "Node 2" (ie that one cited in the setups above as "_the last server_"), will be configured as a pure "**arbiter**" (ie it does not handle/replicate data but only metadata for all files) for all Gluster volumes, thus allowing to use a less "performant" machine, disk/CPU-wise, without impacting global performances; we nonetheless currently support the initial creation of an infrastructure based on **1 server machine** (needing at least oVirt version 4.2) while we expect to support in the near future **4 and more server machines** and also the more delicate issue of expanding an existing setup, eg from 1-server-machine to 3-server-machines or from 3-server-machines to 4-or-more-server-machines.

We must also stress that, as a further form of provocation/_heresy_, our project uses (always created and published as noted above):  

* the **Gluster/Samba/Ganesha packages** not from the community version, but **rebuilt from RHGS sources** ([RHGS][17] is the downstream Gluster equivalent sold by Red Hat bundled with a support contract) in order to extend the supported timeframe of each version
* the **Openvswitch packages from RDO and OVN packages rebuilt from more recent sources**
  
  

##  Conclusions

  
Concluding we note that the project intends to work on the following pending points, in order of relevance:

1. **Document what we did**, not only in form of comments inside scripts/Kickstarts and of this general overview, but also by adding a quickstart guide and a detailed technical guide;
2. **Improve the Ansible/gDeploy part** (now living in the [appropriate Github repository][64]) in order to make it generally useful (ideally not strictly depending on the Kickstart-based installations) and adhering to general standards for playbooks/modules;
3. Investigate the feasibility and community interest for an **oVirt packages rebuild** project, not from community sources but **from RHV source packages** ([RHV][69] is the downstream oVirt equivalent sold by Red Hat bundled with a support contract) in order to extend the supported timeframe of each version.
  
Among further future developments (some of which have been already started) there are also integrated solutions for:  

* **backup** (using the [Bareos][27] free software, recompiled from source in order to extend the supported timeframe of each version);
* **monitoring** (centralized gathering of logs and/or integration with the "Metrics Store" solution [being developed for oVirt][28]);
* **management/orchestration/provisioning** (based on the [Foreman][66] free software in a "[Red Hat Satellite][67]-like" configuration, both substituting our current PC/VD support machine and as a living-on hosted vm);
* **automated complete infrastructure shutdown** unattended and driven by external events (just think of small setups with a single UPS or even an enterprise UPS but without the guarantee of uninterrupted power supply from emergency generators).
  
  

[10]: https://www.ovirt.org/
[11]: https://www.redhat.com/en/technologies/virtualization
[12]: https://www.centos.org/
[13]: https://www.redhat.com/it/technologies/linux-platforms/enterprise-linux
[14]: https://www.ovirt.org/node/
[15]: https://access.redhat.com/documentation/en-us/red_hat_virtualization/4.1/html/installation_guide/red_hat_virtualization_hosts
[16]: https://www.gluster.org/
[17]: https://www.redhat.com/en/technologies/storage/gluster
[18]: http://openvswitch.org/support/dist-docs/ovn-architecture.7.html
[19]: https://ctdb.samba.org/
[20]: https://www.samba.org/
[21]: https://github.com/nfs-ganesha/nfs-ganesha/wiki
[24]: http://cockpit-project.org/
[25]: https://dangerous.ovirt.life/hvp-repos/el7/
[26]: https://dangerous.ovirt.life/hvp-repos/RPM-GPG-KEY-hvp
[27]: https://www.bareos.org/
[28]: http://www.ovirt.org/develop/release-management/features/metrics/metrics-store/
[60]: https://www.ovirt.org/documentation/self-hosted/Self-Hosted_Engine_Guide/
[61]: https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/installation_guide/chap-kickstart-installations
[62]: https://github.com/gluster/gdeploy
[63]: https://www.ansible.com/
[64]: https://github.com/Heretic-oVirt/ansible
[65]: https://github.com/gluster/gluster-block
[66]: https://www.theforeman.org/
[67]: https://access.redhat.com/products/red-hat-satellite
[68]: https://wiki.x2go.org/
[69]: https://www.redhat.com/en/technologies/virtualization/enterprise-virtualization
[70]: https://en.wikipedia.org/wiki/Software-defined_data_center
