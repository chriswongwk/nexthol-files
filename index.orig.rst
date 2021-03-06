.. title:: Nutanix .Next Files HOL

.. toctree::
  :maxdepth: 2
  :caption: Appendix
  :name: _appendix
  :hidden:

  tools_vms/linux_tools_vm_cloud-init
  tools_vms/windows_tools_vm

.. _files:

-----
Files
-----

*The estimated time to complete this lab is 60 minutes.*

Overview
++++++++

Traditionally, file storage has been yet another silo within IT, introducing unnecessary complexity and suffering from the same issues of scale and lack of continuous innovation seen in SAN storage. Nutanix believes there is no room for silos in the Enterprise Cloud. By approaching file storage as an app, running in software on top of a proven HCI core, Nutanix Files  delivers high performance, scalability, and rapid innovation through One Click management.

**In this lab you will step through a Files deployment, manage SMB shares and NFS exports, scale out the environment, and explore upcoming Files features. The lab will provide key considerations around deployment, configuration, and use cases.**


Lab Setup
+++++++++

This lab requires applications provisioned as part of the :ref:`windows_tools_vm`.

If you have not yet deployed this VM, see the linked steps and deploy this VM after starting the installation of files in the next section.

Deploying Files
+++++++++++++++
.. note::

  In the interest of saving time, the Files 3.5 package has already been uploaded to your cluster and a File Server has been deployed with the name of "HOLFS". Files binaries can be downloaded directly through Prism or uploaded manually. To deploy Files on a cluster, you would navigate to **File Server** in the main Prism menu, and select **+ File Server**:

  .. figure:: images/1.png

  You would then follow the on-screen steps to define a name for the file server, along with network configuration settings. To see the settings with which the file server has been deployed, navigate to **File Server** in the Prism Element menu, then click on the file server "HOLFS," then select **Update > Network Configuration** to see the file server network configuration settings


Enabling Protocols for the File Server
..................

By default, the file server deployed has SMB enabled but not NFS.  Before we begin deploying NFS exports, we need to enable NFS Protocol on the File Server.

#. In **Prism > File Server**, click **Protocol Management**, then click **Directory Management**

#. Click tick box "Use NFS Protocol"

   .. figure:: images/2.png

#. Then select "Update". This will now enable you to deploy both SMB and NFS shares


Using SMB Shares
++++++++++++++++

In this exercise you will create and test a SMB share, used to support home directories, user profiles, and other unstructured file data such as departmental shares commonly accessed by Windows clients.

Creating the Share
..................

#. In **Prism > File Server**, click **+ Share/Export**.

#. Fill out the following fields:

   - **Name** - Marketing
   - **Description (Optional)** - Departmental share for marketing team
   - **File Server** - **HOLFS**
   - **Share Path (Optional)** - Leave blank. This field allows you to specify an existing path in which to create the nested share.
   - **Max Size (Optional)** - Leave blank. This field allows you to set a hard quota for the individual share.
   - **Select Protocol** - SMB

   .. figure:: images/14.png

#. Click **Next**.

#. Select **Enable Access Based Enumeration** and **Self Service Restore**.

   .. figure:: images/15.png

   Because this is a single node AOS cluster and therefore a single file server VM, all shares will be **Standard** shares. A Standard share means that all top level directories and files within the share, as well as connections to the share, are served from a single file server VM.

   If this were a three node Files cluster or larger you’d have an option to create a **Distributed** share.  Distributed shares are appropriate for home directories, user profiles, and application folders. This type of share shards top level directories across all Files VMs and load balances connections across all Files VMs within the Files cluster.

   **Access Based Enumeration (ABE)** ensures that only files and folders which a given user has read access are visible to that user. This is commonly enabled for Windows file shares.

   **Self Service Restore** allows users to leverage Windows Previous Version to easily restore individual files to previous revisions based on Nutanix snapshots.

#. Click **Next**.

#. Review the **Summary** and click **Create**.

   .. figure:: images/16.png

Testing the Share
.................

#. Connect to your *Initials*\ **-ToolsVM** via RDP or console.

   .. note::

     The Tools VM has already been joined to the **NTNXLAB.local** domain. You could use any domain joined VM to complete the following steps.

#. Open ``\\HOLFS.ntnxlab.local\`` in **File Explorer**. If prompted for credentials, use the Domain Administrator account:
    - administrator@ntnxlab.local
    - nutanix/4u

   .. figure:: images/17.png

#. Test accessing the Marketing share by creating a text file or copying a file created on the Tools VM to the share.


   - The **NTNXLAB\\Administrator** user was specified as a Files Administrator during deployment of the Files cluster, giving it read/write access to all shares by default.
   - Managing access for other users is no different than any other SMB share.

#. Right-click **Marketing > Properties**.

#. Select the **Security** tab and click **Advanced**.

   .. figure:: images/19.png

#. Select **Users (**\ *HOLFS*\ **-Files\\Users)** and click **Remove**.

#. Click **Add**.

#. Click **Select a principal** and specify **Everyone** in the **Object Name** field. Click **OK**.

   .. figure:: images/20.png

#. Fill out the following fields and click **OK**:

   - **Type** - Allow
   - **Applies to** - This folder only
   - Select **Read & execute**
   - Select **List folder contents**
   - Select **Read**
   - Select **Write**

   .. figure:: images/21.png

#. Click **OK > OK > OK** to save the permission changes.

   All users will now be able to create folders and files within the Marketing share.

   It is common for shares utilized by many people to leverage quotas to ensure fair use of resources. Files offers the ability to set either soft or hard quotas on a per share basis for either individual users within Active Directory, or specific Active Directory Security Groups.

#. In **Prism > File Server > Share > Marketing**, click **+ Add Quota Policy**.

#. Fill out the following fields and click **Save**:

   - Select **Group**
   - **User or Group** - SSP Developers
   - **Quota** - 10 GiB
   - **Enforcement Type** - Hard Limit

   .. figure:: images/22.png

#. Click **Save**.

#. With the Marketing share still selected, review the **Share Details**, **Usage** and **Performance** tabs to understand the available on a per share basis, including the number of files & connections, storage utilization over time, latency, throughput, and IOPS.

   .. figure:: images/23.png

Using NFS Exports
+++++++++++++++++

In this exercise you will create and test a NFSv4 export, used to support clustered applications, store application data such as logging, or storing other unstructured file data commonly accessed by Linux clients.

Creating the Export
...................

#. In **Prism > File Server**, click **+ Share/Export**.

#. Fill out the following fields:

   - **Name** - logs
   - **Description (Optional)** - File share for system logs
   - **File Server** - HOLFS**
   - **Share Path (Optional)** - Leave blank
   - **Max Size (Optional)** - Leave blank
   - **Select Protocol** - NFS

   .. figure:: images/24.png

#. Click **Next**.

#. Fill out the following fields:

   - Select **Enable Self Service Restore**
      - These snapshots appear as a .snapshot directory for NFS clients.
   - **Authentication** - System
   - **Default Access (For All Clients)** - No Access
   - Select **+ Add exceptions**
   - **Clients with Read-Write Access** - *The first 3 octets of your cluster network*\ .* (e.g. 10.38.1.\*)

   .. figure:: images/25.png

   By default an NFS export will allow read/write access to any host that mounts the export, but this can be restricted to specific IPs or IP ranges.

#. Click **Next**.

#. Review the **Summary** and click **Create**.

Testing the Export
..................

You will first provision a CentOS VM to use as a client for your Files export.

.. note:: If you have already deployed the :ref:`linux_tools_vm` as part of another lab, you may use this VM as your NFS client instead.

#. In **Prism > VM > Table**, click **+ Create VM**.

#. Fill out the following fields:

   - **Name** - *Initials*\ -NFS-Client
   - **Description** - CentOS VM for testing Files NFS export
   - **vCPU(s)** - 2
   - **Number of Cores per vCPU** - 1
   - **Memory** - 2 GiB
   - Select **+ Add New Disk**
      - **Operation** - Clone from Image Service
      - **Image** - CentOS
      - Select **Add**
   - Select **Add New NIC**
      - **VLAN Name** - Primary
      - Select **Add**

#. Click **Save**.

#. Select the *Initials*\ **-NFS-Client** VM and click **Power on**.

#. Note the IP address of the VM in Prism, and connect via SSH using the following credentials:

   - **Username** - root
   - **Password** - nutanix/4u

#. Execute the following:

     .. code-block:: bash

       [root@CentOS ~]# yum install -y nfs-utils #This installs the NFSv4 client
       [root@CentOS ~]# mkdir /filesmnt
       [root@CentOS ~]# mount.nfs4 HOLFS.ntnxlab.local:/ /filesmnt/
       [root@CentOS ~]# df -kh
       Filesystem                      Size  Used Avail Use% Mounted on
       /dev/mapper/centos_centos-root  8.5G  1.7G  6.8G  20% /
       devtmpfs                        1.9G     0  1.9G   0% /dev
       tmpfs                           1.9G     0  1.9G   0% /dev/shm
       tmpfs                           1.9G   17M  1.9G   1% /run
       tmpfs                           1.9G     0  1.9G   0% /sys/fs/cgroup
       /dev/sda1                       494M  141M  353M  29% /boot
       tmpfs                           377M     0  377M   0% /run/user/0
       *intials*-Files.ntnxlab.local:/             1.0T  7.0M  1.0T   1% /afsmnt
       [root@CentOS ~]# ls -l /filesmnt/
       total 1
       drwxrwxrwx. 2 root root 2 Mar  9 18:53 logs

#. Observe that the **logs** directory is mounted in ``/filesmnt/logs``.

#. Reboot the VM and observe the export is no longer mounted. To persist the mount, add it to ``/etc/fstab`` by executing the following:

     .. code-block:: bash

       echo 'Intials-Files.ntnxlab.local:/ /filesmnt nfs4' >> /etc/fstab

#. The following command will add 100 2MB files filled with random data to ``/filesmnt/logs``:

     .. code-block:: bash

       mkdir /filesmnt/logs/host1
       for i in {1..100}; do dd if=/dev/urandom bs=8k count=256 of=/filesmnt/logs/host1/file$i; done

#. Return to **Prism > File Server > Share > logs** to monitor performance and usage.

   Note that the utilization data is updated every 10 minutes.

File Analytics
+++++++++++++++++

In this exercise you will deploy the File Analytics VM and scan the existing shares to build out the dashboard.  You will also create anomaly alerts and view the audit details for your file server instance.

#. In **Prism** > **File Server** > click **Deploy File Analytics**

   .. figure:: images/31.png

#. Select **Deploy**

#. Choose **Download** for the 2.0.x version available

.. note::

   For the purpose of saving time, the Files 3.5 package has already been uploaded to your cluster. Files binaries can be downloaded directly through Prism or uploaded manually.

#. Fill out the details

   - **Name** - <File Server Name>
   - **Storage Container** – Will automatically select the container used by your file server instance
   - **Network List** – Primary - Managed

#. Select **Show Advanced Settings**

#. Ensure **DNS Resolver IP** is set to your Active Directory, ntnxlab.local, domain controller/DNS IP address and **ONLY** that address.

    .. figure:: images/FA001.png

#. Choose **Deploy**

#. You can monitor the deployment from the **Tasks** page.  The Analytics VM deployment should take ~5 minutes.

#. In **Prism** > **File Server** > click **File Analytics**

   .. figure:: images/33.png

#. On the Enable File Analytics page enter your domain administrator which is also your file server administrator.

   - **Username**: administrator
   - **Password**: nutanix/4u

   .. figure:: images/34.png

#. Select **Enable**

#. Analytics will perform an initial scan of the existing shares which will take just a couple minutes.  You can see the scan by going to the gear icon within the Analytics UI and selecting **Scan File System**

   .. figure:: images/35.png

#. Choose **Cancel** to exit the scan details window

#. After viewing the scan details, refresh your browser.  You should see the **Data Age**, **File Distribution by Size** and **File Distribution by Type** dashboard panels update.

   .. figure:: images/36.png

#. Create some audit trail activity by going to the marketing share and opening one of the word files under **Sample Data** > **Documents**

   .. note:: You may need to complete a short wizard for OpenOffice if using that application to open a file.

#. Refresh the **Dashboard** page in your browser to see the **Top 5 active users**, **Top 5 accessed files** and **File Operations** panels update

   .. figure:: images/37.png

#. Click on your user under **Top 5 active users**.  This will take you to the audit trail of the user.

#. You can also click on the **Audit Trails** menu and search for either your user or a given file.  You can use wildcards for your search, for example **.doc**

   .. figure:: images/38.png

#. Next, create two anomaly rules by going to **Define Anomaly Rules** from under the gear icon

   .. figure:: images/39.png

#. Choose **Define Anomaly Rules** and create a rule with the following settings

   - **Events:** Delete
   - **Minimum Operation %:** 1
   - **Minimum Operation Count:** 10
   - **User:** All Users
   - **Type:** Hourly
   - **Interval:** 1

   .. note::

     The minimum operations percentage is calculated based on the number of files. For example, if there are 100 files, and the minimum operations percentage is set to 5, 5 operations within the scan interval would trigger an anomaly alert.

     Minimum Operation Count: Enter a value for a minimum operation threshold.
     Analytics triggers an anomaly alert after meeting the threshold.

#. Choose **Save** for that anomaly table entry

#. Choose **+ Configure new anomaly** and create a second rule with the following settings

   - **Events**: Create
   - **Minimum Operation %**: 1
   - **Minimum Operation Count**: 10
   - **User**: All Users
   - **Type**: Hourly
   - **Interval**: 1

#. Choose **Save** for that anomaly table entry

   .. figure:: images/40.png

#. Select **Save** to exit the Define Anomaly Rules window

#. Go to the Sample Data folder in the Marketing share and copy, then paste that folder to the same share.

   .. figure:: images/42.png

#. Now delete the original Sample Data folder.

#. While waiting for the Anomaly Alerts to populate we’ll create a permission denial.

   .. note:: The Anomaly engine runs every 30 minutes.  While this setting is configurable from the File Analytics VM, modifying this variable is outside the scope of this lab.

#. Create a new directory called **RO** in the Marketing share

#. Create a text file in the **RO** directory with some text like “hello world” called **myfile.txt**

#. Go to the **Properties** of the **RO** folder and select the Security tab

#. Select **Advanced**

#. Choose **Disable inheritance** and select the **Convert…** option

#. Then add the **Everyone** permissions with the following:

   - Read & Execute
   - List folder contents
   - Read

   .. figure:: images/43.png

#. Choose **OK** then **OK** again

#. Open a PowerShell window as a specific user

   - Hold down **Shift** and **right click** on the **PowerShell icon** on the taskbar
   - Select **Run as different user**

   .. figure:: images/44.png

#. Enter the following

   - **User name**: Poweruser01
   - **Password**: nutanix/4u

#. Change Directories into the Marketing share and the **RO** directory

     .. code-block:: bash

        cd \\xyz-files.ntnxlab.local\marketing\RO

#. Execute the following commands, the first should succeed, the second should fail:

     .. code-block:: bash

        more .\myfile.txt
        rm .\myfile.txt

   .. figure:: images/45.png

#. After a minute or so you should see **Permission Denials** in both the dashboard and the **Audit Trails** view.  You may need to refresh your browser.

   .. figure:: images/46.png

   .. note:: The Capacity Trend dashboard panel updates every 24 hrs.

New with Files 3.5
++++++++++++

With the recent Files 3.5 release we have introduced:

- Support for NFSv3
- Support for Self-Service File Restore for NFS (currently supported for SMB shares)
- Support for Change File Tracking (CFT) Backup for NFS (currently supported for SMB shares)
- Support for Nutanix software-based Data-At-Rest Encryption
- Support for multi-protocol access to shares (Now GA with Files 3.5.1)
- File Analytics, a comprehensive view into Files usage for the purposes of insights into file system data, file and user audit trails and anomaly detection (Now GA with Files 3.5.2 and File Analytics 2.0)


Takeaways
+++++++++

What are the key things you should know about **Nutanix Files**?

- Files can be rapidly deployed on top of existing Nutanix clusters, providing SMB and NFS storage for user shares, home directories, departmental shares, applications, and any other general purpose file storage needs.
- Files is not a point solution. VM, File, Block, and Object storage can all be delivered by the same platform using the same management tools, reducing complexity and management silos.
- Files can scale up and scale out with One Click performance optimization.
- File Analytics helps you better understand how data is utilized by your organizations to help you meet your data auditing, data access minimization and compliance requirements.
