.. title:: Nutanix Flow Bootcamp

.. toctree::
   :maxdepth: 2
   :caption: Flow
   :name: _flow
   :hidden:

   .. labsetup/labsetup
   flow_secure_fiesta/flow_secure_fiesta
   beam_security_compliance/beam_security_compliance
   flow_next/flow_next

.. toctree::
  :maxdepth: 2
  :caption: Optional Labs
  :name: _optional_labs
  :hidden:

  flow_assign_categories_in_calm/flow_assign_categories_in_calm
  flow_isolate_fiesta/flow_isolate_fiesta
  security/security

.. toctree::
  :maxdepth: 2
  :caption: Appendix
  :name: _appendix
  :hidden:

  tools_vms/windows_tools_vm
  tools_vms/linux_tools_vm
  appendix/glossary

.. _getting_started:

---------------
Getting Started
---------------

Welcome to the Nutanix Enterprise Private Cloud bootcamp. This workbook accompanies an instructor-led session that introduces Nutanix core technologies, and many common management tasks.

You will explore:

   - Prism Central - Become familiar with its features and navigation.
   - Prism - Perform basic cluster administration tasks, including storage and networking.
   - Walk through basic VM deployment and management tasks.
   - VM data protection, including snapshots and replication.

The instructor explains the exercises, and answers any additional questions that you may have.

At the end of the bootcamp, attendees should understand the core concepts and technologies that make up the Nutanix Enterprise Cloud stack, and should be well prepared for a proof-of-concept (POC) engagement.

What's New
++++++++++

- Workshop updated for the following software versions:
    - AOS 5.19.2
    - Prism 2021.3.0.1

.. - Optional Lab Updates:

Agenda
++++++

- Introductions
- Nutanix Presentation
- A Day in the Life
- Prism Ops
- Platform Security
- Files
- Flow

Introductions
+++++++++++++

- Name
- Familiarity with Nutanix

Initial Setup
+++++++++++++

- Take note of the *Passwords* being used.
- Log into your virtual desktops (connection info below)

Cluster assignment
++++++++++++++++++

The instructor will tell the attendees their assigned clusters.

   .. note::

      If these are Single-Node Clusters (SNCs) pay close attention on the networking portion. The SNCs are configured differently compared to three- or four-node clusters.

Environment Details
+++++++++++++++++++

Nutanix bootcamps are intended to be run in the Nutanix Hosted POC (HPOC) environment. Your cluster will be provisioned with all necessary images, networks, and VMs required to complete the exercises.

Networking
..........

As we are able to provide three/four node clusters and single node clusters in the HPOC environment, we need to describe each sort of cluster separately. The clusters are setup and configured differently.

Three/Four node HPOC clusters
-----------------------------

Three- or four-node Hosted POC clusters follow a standard naming convention:

- **Cluster Name** - POC\ *XYZ*
- **Subnet** - 10.\ **21**\ .\ *XYZ*\ .0
- **Cluster IP** - 10.\ **21**\ .\ *XYZ*\ .37

For example:

- **Cluster Name** - POC055
- **Subnet** - 10.38.55.0
- **Cluster IP** - 10.21.55.37 for the VIP of the Cluster


Throughout the bootcamp, there are multiple instances where you will need to substitute *XYZ* with the correct octet for your subnet, for example:

.. list-table::
  :widths: 25 75
  :header-rows: 1

  * - IP Address
    - Description
  * - 10.38.\ *XYZ*\ .37
    - Nutanix Cluster Virtual IP
  * - 10.38.\ *XYZ*\ .39
    - **PC** VM IP, Prism Central
  * - 10.38.\ *XYZ*\ .41
    - **AutoAD** VM IP, NTNXLAB.local Domain Controller

Each cluster is configured with 2 VLANs which can be used for VMs:

.. list-table::
  :widths: 25 25 10 40
  :header-rows: 1

  * - Network Name
    - Address
    - VLAN
    - DHCP Scope
  * - Primary
    - 10.38.\ *XYZ*\ .1/25
    - 0
    - 10.38.\ *XYZ*\ .50-10.21.\ *XYZ*\ .124
  * - Secondary
    - 10.38.\ *XYZ*\ .129/25
    - *XYZ1*
    - 10.38.\ *XYZ*\ .132-10.21.\ *XYZ*\ .253

Single Node HPOC Clusters
-------------------------

For some bootcamps, we are using Single Node Clusters (SNC). The reason for this is to allow more people to have a dedicated cluster but still have enough free clusters for the larger bootcamps, including those for customers.

The network in the SNC config is using a /26 network prefix length. This splits the network address into four equal sizes that can be used for bootcamps. The below table describes the setup of the network in the four partitions. It provides essential information for the bootcamp with respect to the IP addresses and the services running at that IP address.

.. list-table::
  :widths: 15 15 15 15 40
  :header-rows: 1

  * - Partition 1
    - Partition 2
    - Partition 3
    - Partition 4
    - Service
    - Comment
  * - 10.38.x.1
    - 10.38.x.65
    - 10.38.x.129
    - 10.38.x.193
    - Gateway
    -
  * - 10.38.x.5
    - 10.38.x.69
    - 10.38.x.133
    - 10.38.x.197
    - AHV Host
    -
  * - 10.38.x.6
    - 10.38.x.70
    - 10.38.x.134
    - 10.38.x.198
    - CVM IP
    -
  * - 10.38.x.7
    - 10.38.x.71
    - 10.38.x.135
    - 10.38.x.199
    - Cluster IP
    -
  * - 10.38.x.8
    - 10.38.x.72
    - 10.38.x.136
    - 10.38.x.200
    - Data Services IP
    -
  * - 10.38.x.9
    - 10.38.x.73
    - 10.38.x.137
    - 10.38.x.201
    - Prism Central IP
    -
  * - 10.38.x.11
    - 10.38.x.75
    - 10.38.x.139
    - 10.38.x.203
    - AutoAD IP(DC)
    -
  * - 10.38.x.32-37
    - 10.38.x.96-101
    - 10.38.x.160-165
    - 10.38.x.224-229
    - Objects 1
    -
  * - 10.38.x.38-58
    - 10.38.x.102-122
    - 10.38.x.166-186
    - 10.38.x.230-250
    - Primary network IPAM
    - 6 Free IPs free for static assignment


Credentials
...........

.. note::

  The *<CLUSTER-PASSWORD>* is unique to each cluster and will be provided by the leader of the Workshop.

.. list-table::
  :widths: 25 35 40
  :header-rows: 1

  * - Credential
    - Username
    - Password
  * - Prism Element
    - admin
    - *<CLUSTER-PASSWORD>*
  * - Prism Central
    - admin
    - *<CLUSTER-PASSWORD>*
  * - Controller VM
    - nutanix
    - *<CLUSTER-PASSWORD>*
  * - Prism Central VM
    - nutanix
    - *<CLUSTER-PASSWORD>*

Each cluster has a dedicated domain controller VM responsible for providing Microsoft Active Directory (AD) services for the **NTNXLAB.local** domain. The domain is populated with the following users and groups:

.. list-table::
  :widths: 25 35 40
  :header-rows: 1

  * - Group
    - Username(s)
    - Password
  * - Administrators
    - Administrator
    - nutanix/4u
  * - SSP Admins
    - adminuser01 - adminuser25
    - nutanix/4u
  * - SSP Developers
    - devuser01 - devuser25
    - nutanix/4u
  * - SSP Power Users
    - poweruser01 - poweruser25
    - nutanix/4u
  * - SSP Basic Users
    - basicuser01 - basicuser25
    - nutanix/4u

Access Instructions
+++++++++++++++++++

The Nutanix Hosted POC environment can be accessed several ways:

Parallels VDI
.............

Login to:

   - `https://xld-uswest1.nutanix.com` (for PHX)
   - `https://xld-useast1.nutanix.com` (for RTP)

**Nutanix Employees** - Use your NUTANIXDC credentials
**Non-Employees** - **Username:** POCxxx-User01 (up to POCxxx-User20), **Password:** *<Provided by Instructor>*

Pulse Secure VPN
................

Download the client:

   - Log in to `https://xlv-uswest1.nutanix.com` or `https://xlv-useast1.nutanix.com` - **Username:** POCxxx-User01 (up to POCxxx-User20), **Password:** *<Provided by Instructor>*

In Pulse Secure Client, **Add** a connection:

For PHX:

- **Type** - Policy Secure (UAC) or Connection Server
- **Name** - X-Labs - PHX
- **Server URL** - xlv-uswest1.nutanix.com

For RTP:

- **Type** - Policy Secure (UAC) or Connection Server
- **Name** - X-Labs - RTP
- **Server URL** - xlv-useast1.nutanix.com
