# Overview

This lab will guide you through the process of enabling and collecting package inventory so that you can easily view an inventory of software packages in Puppet Enterprise.

You'll perform the following steps:

* Enable Package Inventory
* Collect Package Inventory

# Steps

### Step 1: Enable Package Inventory

1. Navigate to the Puppet console at `https://<your-puppet-server-hostname>` and login with the **admin** user and the password you generated earlier.
2. From the navigation bar on the left, click on **Node groups**.
3. Expand the **PE infrastructure** by clicking the **⊞** sign then click the **PE Agent** node group.
4. Navigate to the **Classes** tab.
5. In the **`puppet_enterprise::profile::agent`** class, click **Parameter name** and select **`package_inventory_enabled`** from the list.
6. Set it's value to **true**. Make sure this value is <span style="text-decoration:underline;">not</span> surrounded by quotes.
7. Click **Add to node group**.
8. To save the changes, click **Commit 1 change** in the lower right hand corner of the screen. 

### Step 2: Collect Package Inventory

We need to perform two Puppet runs to begin collecting package inventory. The first run will apply the setting to your nodes and _enable_ package inventory collection. The second run will _collect_ package inventory for the first time.



1. From the **PE Agent** node group, in the top right corner click **Run > Puppet**.
2. Once in the **Run Puppet** screen, click the **Run job** button on the bottom right hand corner of the screen - Wait for the Puppet run to complete.
3. Once the Puppet run is complete, click the **Run again > All nodes** button on the top right hand corner of the screen and click **Run job** again.
4. In the navigation bar under **INVENTORY**, click **Packages**.
   
You should now find an inventory of packages, their versions, providers and the amount of instances across the estate.