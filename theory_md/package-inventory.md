<div class="tocoutline">

### Table of Contents

<div class="toc">

- [Overview](#overview)
    - [Viewing Package inventory](#viewing-package-inventory)
    - [Enable and collect Package inventory](#enable-and-collect-package-inventory)

</div>

</div>

# Overview<a href="#overview" aria-hidden="true"></a>

Puppet can automatically gather package information across your infrastructure such as package name, version, package manager. Package inventory makes it easy to see which packages you have across your estate without any extra effort on your part. Package inventory is updated each time the Puppet agent reaches out to the Primary Puppet server (default is every 30 minutes).

### Viewing Package inventory<a href="#viewing-package-inventory" aria-hidden="true"></a>

Before viewing package inventory, it must first be enabled. Once enabled, you can navigate to the **Packages** page in the PE sidebar under **Inventory**. You can find an Inventory of packages, their versions, providers and amount of instances across the estate. 

![alt text for screen readers](../assets/img/package_inventory.png "Package Inventory")

### Enable and collect Package inventory<a href="#enable-and-collect-package-inventory" aria-hidden="true"></a>

To collect package inventory, you’ll need to enable it first for all agents. You’ll then need to perform two Puppet runs to begin collecting package inventory. The first run will apply the setting to your nodes and _enable_ package inventory collection. The second run will _collect_ package inventory for the first time.


1. Open the Puppet Enterprise Web Console at <span style="text-decoration:underline;">https://&lt;IP address or FQDN></span> and login with username **admin** and the password you set earlier.
2. Under Inventory on the side bar, click the Node groups
3. Expand the **PE infrastructure** by clicking the **⊞** sign, then click the **PE Agent** node group.
4. Navigate to the **Classes** tab
5. In the **puppet_enterprise::profile::agent** class, click **Parameter name** and select **package_inventory_enabled** from the list
6. Set its **Value** to **true**.
7. Click **Add to node group**
8. To save the changes, click **Commit 1 change** in the lower right hand corner of the screen. 
9. Ensure you’re still within the **PE Agent** node group and in the top right corner click **Run > Puppet**
10. Once in the **Run Puppet** screen, click the **Run job** button on the bottom right hand corner of the screen - Wait for the Puppet run to complete.
11. Once the Puppet run is complete, click the **Run again** button on the top right hand corner of the screen and click **Run job** again


You can now view package information across your nodes within PE.
