.. title:: Nutanix .Next Flow HOL


.. _flow:

----
Flow
----

*The estimated time to complete this lab is 90 minutes.*

Overview
++++++++

Flow is a software-defined networking product tightly integrated into Nutanix AHV and Prism Central. Flow provides rich visualization, automation, and security for VMs running on AHV.
Microsegmentation is a component of Flow that simplifies policy management. Using multiple Prism Central categories (logical groups), you can create a powerful distributed firewall that gives administrators an application-centric policy management tool for securing VM traffic.
Combining this with Calm allows automated deployment of applications that are secured as they are created.

**In this lab you will use Flow to build a microsegmentation policy for a multi-tier web application, isolate groups of VMs from one another, and quarantine a problematic VM.**

Lab Setup
+++++++++

This lab depends on the availability of a multi-tier **Task Manager** web application.

Refer to the :ref:`taskman` lab for instructions on importing and launching the completed **Task Manager** blueprint.

.. note::

  Even if you completed the **Calm: Linux Workloads** lab, please import the above **Task Manager** blueprint.

Once you have initiated the **Task Manager** deployment, you can proceed with the lab below. **You do not need to wait for the blueprint deployment to complete to begin this lab.**

Enabling Flow
+++++++++++++

Flow is built into Prism Central and requires no additional appliances or consoles to manage. Before you can begin securing your environment with Flow, the service must be enabled.

.. note::

  Flow can only be enabled once per Prism Central instance. Flow will require an additional 1GB of memory for each Prism Central VM, but there is no action required by the user as this occurs automatically. Additionally, the enable process verifies that each connected AHV host has at least 1GB of free RAM. The list of Flow capable AHV clusters is listed in the enable window.

  If **Microsegmentation** displays a green check mark next to it, that means Flow has already been enabled for the Prism Central instance being used. Proceed to `Securing An Application`_.

#. In **Prism Central**, click the **?** drop down menu and select **Microsegmentation**.

   .. figure:: images/10.png

   Enabling Flow will require an additional 1GB of memory for each Prism Central VM, but there is no action required by the user as this occurs automatically.

#. Click Microsegmentation. The Enable Microsegmentation dialog box is displayed. Select the Enable Microsegmentation check box. Click OK.


Securing An Application
+++++++++++++++++++++++

Flow provides multiple System categories out of the box, such as AppType, AppTier, and Environment, that are used to quickly group virtual machines. Security policies are applied using these VM groups. Start using these pre-existing categories right away, or add your own categories for custom grouping.

Defining Category Values
........................

Prism Central uses categories as metadata to tag VMs to determine how policies will be applied.

#. In **Prism Central**, select :fa:`bars` **> Virtual Infrastructure > Categories**.

#. Select the checkbox for **AppType** and click **Actions > Update**.

   .. figure:: images/12.png

#. Click the :fa:`plus-circle` icon beside the last value to add an additional Category value.

#. Specify *Initials*-**TaskMan**  as the value name.

   .. figure:: images/13.png

#. Click **Save**.

#. Select the checkbox for **AppTier** and click **Actions > Update**.

#. Click the :fa:`plus-circle` icon beside the last value to add an additional Category value.

#. Specify *Initials*-**TMWeb**  as the value name. This category will be applied to the application's web tier.

#. Click :fa:`plus-circle` and specify *Initials*-**TMDB**. This category will be applied to the application's MySQL database.

#. Click :fa:`plus-circle` and specify *Initials*-**TMLB**. This category will be applied to the application's HAProxy load balancer.

   .. figure:: images/14.png

#. Click **Save**.

Creating a Security Policy
..........................

Nutanix Flow includes a policy-driven security framework that uses a workload-centric approach instead of a network-centric approach. Therefore, it can scrutinize traffic to and from VMs no matter how their network configurations change and where they reside in the data center. The workload-centric, network-agnostic approach also enables the virtualization team to implement these security policies without having to rely on network security teams.

Security policies are applied to categories (a logical grouping of VMs) and not to the VMs themselves. Therefore, it does not matter how many VMs are started up in a given category. Traffic associated with the VMs in a category is secured without administrative intervention, at any scale.

While you wait for the Task Manager application to be deployed from the Calm blueprint, create the security policies that will protect the application.

#. In **Prism Central**, select :fa:`bars` **> Policies > Security Policies**.

#. Click **Create Security Policy > Secure Applications (App Policy) > Create**.

#. Fill out the following fields:

   - **Name** - *Initials*-AppTaskMan
   - **Purpose** - Restrict unnecessary access to Task Manager
   - **Secure this app** - AppType: *Initials*-TaskMan
   - Do **NOT** select **Filter the app type by category**.

   .. figure:: images/18.png

#. Click **Next**.

#. If prompted, click **OK, Got it!** on the tutorial diagram of the **Create App Security Policy** wizard.

#. To allow for more granular configuration of the security policy, click **Set rules on App Tiers** rather than applying the same rules to all components of the application.

   .. figure:: images/19.png

#. Click **+ Add Tier**.

#. Select **AppTier:**\ *Initials*-**TMLB** from the drop down.

#. Repeat Steps 7-8 for **AppTier:**\ *Initials*-**TMWeb** and **AppTier:**\ *Initials*-**TMDB**.

   .. figure:: images/20.png

   Next you will define the **Inbound** rules, which control which sources you will allow to communicate with your application. You can allow all inbound traffic, or define whitelisted sources. By default, the security policy is set to deny all incoming traffic.

   In this scenario we want to allow inbound TCP traffic on port 80 from all clients on the production network.

#. Under **Inbound**, click **+ Add Source**.

#. Start typing **Environment:Production** and click **Add**.

   .. note::

     Sources can also be specified by IP or subnet, but Categories allow for greater flexibility as this data can follow a VM regardless of changes to its network location.

#. To create an inbound rule, select the **+** icon that appears to the left of **AppTier:**\ *Initials*-**TMLB**.

   .. figure:: images/21.png

#. Fill out the following fields:

   - **Protocol** - TCP
   - **Ports** - 80

   .. figure:: images/22.png

   .. note::

     Multiple protocols and ports can be added to a single rule.

#. Click **Save**.

   Calm could also require access to the VMs for workflows including scaling out, scaling in, or upgrades. Calm communicates with these VMs via SSH, using TCP port 22.

#. Under **Inbound**, click **+ Add Source**.

#. Fill out the following fields:

   - **Add source by:** - Select **Subnet/IP**
   - Specify *Your Prism Central IP*\ /32

   .. note::

     The **/32** denotes a single IP as opposed to a subnet range.

   .. figure:: images/23.png

#. Click **Add**.

#. Select the **+** icon that appears to the left of **AppTier:**\ *Initials*-**TMLB**, specify **TCP** port **22** and click **Save**.

#. Repeat Step 18 for **AppTier:**\ *Initials*-**TMWeb** and **AppTier:**\ *Initials*-**TMDB** to allow Calm to communicate with the web tier and database VMs.

   .. figure:: images/24.png

   By default, the security policy allows the application to send all outbound traffic to any destination. The only outbound communication required for your application is for the database VM to be able to communicate with your DNS server.

#. Under **Outbound**, select **Whitelist Only** from the drop down menu, and click **+ Add Destination**.

#. Fill out the following fields:

   - **Add source by:** - Select **Subnet/IP**
   - Specify *Your Domain Controller IP*\ /32

   .. figure:: images/25.png

#. Click **Add**.

#. Select the **+** icon that appears to the right of **AppTier:**\ *Initials*-**TMDB**, specify **UDP** port **53** and click **Save** to allow DNS traffic.

   .. figure:: images/26.png

   Each tier of the application communicates with other tiers and the policy must allow this traffic. Some tiers such as the load balancer and web do not require communication within the same tier.

#. To define intra-app communication, click **Set Rules within App**.

   .. figure:: images/27.png

#. Click **AppTier:**\ *Initials*-**TMLB** and select **No** to prevent communication between VMs in this tier. There is only a single load balancer VM within the tier.

#. While **AppTier:**\ *Initials*-**TMLB** is still selected, click the :fa:`plus-circle` icon to the right of **AppTier:**\ *Initials*-**TMWeb** to create a tier to tier rule.

#. Fill out the following fields to allow communication on TCP port 80 between the load balancer and web tiers:

   - **Protocol** - TCP
   - **Ports** - 80

   .. figure:: images/28.png

#. Click **Save**.

#. Click **AppTier:**\ *Initials*-**TMWeb** and select **No** to prevent communication between VMs in this tier. While there are multiple web server VMs, they do not need to communicate with each other.

#. While **AppTier:**\ *Initials*-**TMWeb** is still selected, click the :fa:`plus-circle` icon to the right of **AppTier:**\ *Initials*-**TMDB** to create another tier to tier rule.

#. Fill out the following fields to allow communication on TCP port 3306 to allow the database connection between the web servers and the MySQL database:

   - **Protocol** - TCP
   - **Ports** - 3306

   .. figure:: images/29.png

#. Click **Save**.

#. Click **Next** to review the security policy.

#. Click **Save and Monitor** to save the policy.

Assigning Category Values
.........................

.. note::

  By this time, your application blueprint should have finished provisioning. If it has not completed, please wait until it has finished to proceed.

You will now apply the previously created categories to the VMs provisioned from your Task Manager blueprint. Flow categories can be assigned as part of a Calm blueprint, but the purpose of this exercise is to understand category assignment to existing virtual machines in an environment.

#. In **Prism Central**, select :fa:`bars` **> Virtual Infrastructure > VMs**.

#. Click **Filters** and search for *Initials-* to display your virtual machines.

   .. figure:: images/15.png

#. Using the checkboxes, select the 4 VMs associated with the application (HAProxy, MYSQL, WebServer-0, WebServer-1) and select **Actions > Manage Categories**.

   .. figure:: images/16.png

   .. note::

     You can also use the **Label** functionality to make searching for this group of VMs faster in the future.

     .. figure:: images/16b.png

#. Specify **AppType:**\ *Initials*-**TaskMan** in the search bar and click **Save** icon to bulk assign the category to all 4 VMs.

#. Select ONLY the *Initials*\ **-HAProxy** VM, select **Actions > Manage Categories**, specify the **AppTier:**\ *Initials*-**TMLB** category and click **Save**.

   .. figure:: images/17.png

#. Repeat Step 5 to assign **AppTier:**\ *Initials*-**TMWeb** to your web tier VMs.

#. Repeat Step 5 to assign **AppTier:**\ *Initials*-**TMDB** to your MySQL VM.

#. Finally, repeat Step 5 to assign **Environment:Dev** to your Windows client VM.

  .. figure:: images/flow-hol-env-dev.png

Monitoring and Applying a Security Policy
+++++++++++++++++++++++++++++++++++++++++

Before applying the Flow policy, you will ensure the Task Manager application is working as expected.

Testing the Application
.......................

#. From **Prism Central > Virtual Infrastructure > VMs**, note the IP address of your *Initials*\ **-HAPROXY-0...** and *Initials*\ **-MYSQL-0...** VMs.

#. Launch the console for your *Initials*\ **-WinClient-0** VM.

   This VM was provisioned as part of the Task Manager application blueprint.

#. From the *Initials*\ **-WinClient-0** console open a browser and access \http://*HAPROXY-VM-IP*/.

#. Verify that the application loads and that tasks can be added and deleted.

   .. figure:: images/30.png

#. Open **Command Prompt** and run ``ping -t MYSQL-VM-IP`` to verify connectivity between the client and database. Leave the ping running.

#. Open a second **Command Prompt** and run ``ping -t HAPROXY-VM-IP`` to verify connectivity between the client and load balancer. Leave the ping running.

   .. figure:: images/31.png

Using Flow Visualization
........................

#. Return to **Prism Central** and select :fa:`bars` **> Virtual Infrastructure > Policies > Security Policies >**\ *Initials*-**AppTaskMan**.

#. Verify that **Environment: Dev** appears as an inbound source. The source and line appear in yellow to indicate that traffic has been detected from your client VM.

   .. figure:: images/32.png

#. Mouse over the line connecting **Environment: Dev** to **AppTier:**\ *Initials*-**TMLB** to view the protocol and connection information.

#. Click the yellow flow line to view a graph of connection attempts over the past 24 hours.

   .. figure:: images/33.png

   Are there any other detected outbound traffic flows? Hover over these connections and determine what ports are in use.

#. Click **Update** to edit the policy.

   .. figure:: images/34.png

#. Click **Next** and wait for the detected traffic flows to populate.

#. Mouse over the **Environment: Dev** source that connects to **AppTier:**\ *Initials*-**TMLB** and click the :fa:`check` icon that appears.

   .. figure:: images/35.png

#. Click **OK** to complete adding the rule.

   The **Environment: Dev** source should now turn blue, indicating that it is part of the policy. Mouse over the flow line and verify that both ICMP (ping traffic) and TCP port 80 appear.

#. Click **Next > Save and Monitor** to update the policy.

Applying Flow Policies
......................

In order to enforce the policy you have defined, the policy must be applied.

#. Select *Initials*-**AppTaskMan**  and click **Actions > Apply**.

   .. figure:: images/36.png

#. Type **APPLY** in the confirmation dialogue and click **OK** to begin blocking traffic.

#. Return to the *Initials*\ **-WinClient-0** console.

   What happens to the continuous ping traffic from the Windows client to the database server? Is this traffic blocked?

#. Verify that the Windows Client VM can still access the Task Manager application using the web browser and the load balancer IP address.

   Can you still enter new tasks that require communication between the web server and database?

Isolating Environments
++++++++++++++++++++++

Isolation policies are used when one group of VMs must be completely blocked from communicating with another group of VMs without any whitelist exceptions. A common example is using isolation policies to block VMs tagged **Environment: Dev** from talking to VMs in **Environment: Production**. Do not use isolation policies if you want to create exceptions between the two groups, instead use an Application Policy which allows a whitelist model.

In this exercise you will create a new environment category and assign this to the Task Manager application. Then you will create and implement an isolation security policy that uses the newly created category in order to restrict unauthorized access.

Creating and Assigning Categories
.................................

#. In **Prism Central**, select :fa:`bars` **> Virtual Infrastructure > Categories**.

#. Select the checkbox for **Environment** and click **Actions > Update**.

#. Click the :fa:`plus-circle` icon beside the last value to add an additional Category value.

#. Specify *Initials*-**Prod** as the value name.

   .. figure:: images/37.png

#. Click **Save**.

#. In **Prism Central**, select :fa:`bars` **> Virtual Infrastructure > VMs**.

#. Click **Filters** and search for *Initials-* to display your virtual machines.

   .. note::

     If you previously created a Label for your application VMs you can also search for that label. Alternatively you can search for the **AppType:** **TaskMan**-*Initials* category from the Filters pane.

     .. figure:: images/38.png

#. Using the checkboxes, select the 4 VMs associated with the application (HAProxy, MYSQL, WebServer-0, WebServer-1) and select **Actions > Manage Categories**.

#. Specify **Environment:**\ *Initials*-**Prod** in the search bar and click **Save** icon to bulk assign the category to all 4 VMs.

   .. figure:: images/39.png

Creating an Isolation Policy
............................

#. In **Prism Central**, select :fa:`bars` **> Virtual Infrastructure > Policies > Security Policies**.

#. Click **Create Security Policy > Isolate Environments (Isolation Policy) > Create**.

#. Fill out the following fields:

   - **Name** - *Initials*-Isolate-dev-prod
   - **Purpose** - *Initials* - Isolate dev from prod
   - **Isolate This Category** - Environment:Dev
   - **From This Category** - Environment:*Initials*-Prod
   - Do **NOT** select **Apply this isolation only within a subset of the datacenter**. This option provides additional granularity by only applying to VMs assigned a third, mutual category.

   .. figure:: images/40.png

#. Click **Apply Now** to save the policy and begin enforcement immediately.

#. Return to the *Initials*\ **-WinClient-0** console.

   Is the Task Manager application accessible? Why not?

   Using these simple policies it is possible to block traffic between groups of VMs such as production and development, to isolate a lab system, or provide isolation for compliance.

Deleting a Policy
.................

#. In **Prism Central**, select :fa:`bars` **> Policies > Security Policies**.

#. Select *Initials*-**Isolate-dev-prod** and click **Actions > Delete**.

#. Type **DELETE** in the confirmation dialogue and click **OK** to disable the policy.

   .. note::

     To disable the policy you can choose to enter **Monitor** mode, rather than deleting the policy completely.

#. Return to the *Initials*\ **-WinClient-0** console and verify the Task Manager application is accessible again from the browser.

Quarantining a VM
+++++++++++++++++

In this task we will place a VM into quarantine and observe the behavior of the VM. We will also inspect the configurable options inside the quarantine policy.

#. Return to the *Initials*\ **-WinClient-0** console.

#. Open a **Command Prompt** and run ``ping -t HAPROXY-VM-IP`` to verify connectivity between the client and load balancer.

   .. note::

     If the ping is unsuccessful you may need to update your Inbound Rule for **Environment:Dev** to **AppTier:**\ *Initials*-**TMLB** to include **Any** as the **Type** and **Code** for **ICMP** traffic as shown below. Apply the updated **AppTaskMan-**\ *Initials* policy and the ping should resume.

     .. figure:: images/41.png

#. In **Prism Central > Virtual Infrastructure > VMs**, select your *Initials*\ **-HAPROXY-0...** VM.

#. Click **Actions > Quarantine VMs**.

   .. figure:: images/42.png

#. Select **Forensic** and click **Quarantine**.

   What happens with the continuous ping between your client and the load balancer? Can you access the Task Manager application web page from the client VM?

#. In **Prism Central**, select :fa:`bars` **> Virtual Infrastructure > Policies > Security Policies > Quarantine** to view all Quarantined VMs.

#. Click **Update** to edit the Quarantine policy.

   To illustrate the capabilities of this special Flow policy, you will add your client VM as a "forensic tool". In production, VMs allowed inbound access to quarantined VMs could be used to run security and forensic suites such as Kali Linux or SANS SIFT.

#. Under **Inbound**, click **+ Add Source**.

#. Fill out the following fields:

   - **Add source by:** - Select **Subnet/IP**
   - Specify *Your WinClient VM IP*\ /32

   To what targets can this source be connected? What is the difference between the Forensic and Strict quarantine mode?

   Note that adding a VM to the **Strict** Quarantine policy disables all inbound and outbound communication to a VM. The **Strict** policy would apply to an VMs whose presence on the network poses a threat to the environment.

#. Click the :fa:`plus-circle` icon to the left of **Quarantine: Forensic** to create an Inbound Rule.

#. Click **Save** to allow any protocol on any port between the client VM and the **Quarantine: Forensic** category.

   .. figure:: images/43.png

#. Click **Next > Apply Now** to save and apply the updated policy.

   What happens to the pings to the load balancer after the source is added? Can you access the Task Manager web application?

#. You can remove the load balancer VM from the **Quarantine: Forensic** category by selecting the VM in Prism Central and clicking **Actions > Unquarantine VMs**.

Takeaways
+++++++++

What are the key things you should know about **Nutanix Flow**?

- In this exercise you utilized Flow to quarantine a VM using the two modalities of the quarantine policy, which are strict and forensic.
- Quarantine policies are evaluated at a higher priority than application policies. A quarantine traffic can block traffic that would otherwise be allowed by an application policy.
- The forensic modality is key to allow limited access a quarantined VM while the VM is quarantined.

.. |blueprints| image:: images/blueprints.png
