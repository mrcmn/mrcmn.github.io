# Overview

In this lab, you'll enable patch management for your lab node by creating a patching node group and assigning the `pe_patch` class to this node group. Your nodes will then start to pull patch information which will be stored within the `pe_patch` fact.

You'll perform the following steps:

* Create patching node group
* Add node to patching group
* Create a pe_patch patching group
* Run Puppet to pull patch information
* Review high level patch information
* Review granular patch information

> PE Patch management assumes that WSUS/Windows update service and relevant Linux package managers such as YUM, APT etc are correctly configured in order for patch management to work as intended.

# Steps

### Step 1: Create patching node group

1. Navigate to the **Node Groups** page and click **Add Group**.
2. Under parent name, Select **PE Patch Management**.
3. For **Group name**, enter **Lab patch group**.
4. Click **Add**.

### Step 2: Add node to patching group

1. Expand the **PE Patch Management** group and click on **Lab patch group**.
2. In the **Rules** tab, under the **Certname** heading, enter the first few characters of your nodes hostname that you installed the agent on earlier, then select that node from the list that appears and click **Pin node**.
3. To save the changes, click **Commit 1 change** in the lower right hand corner of the screen.
4. Verify that your node has been successfully added to the group by reviewing the **Matching nodes** tab.

### Step 3: Create a pe_patch patching group

1. Click on the **classes** tab.
2. To the right of **Add new class** type **pe_patch** then select the class from the dropdown. 
3. Click **Add class**.
4. From the **Parameter name** drop down, select **patch_group** and in the **value** section, enter the name of your patch group - **“Patch Group 1”**.
5. Once complete, click **Add to Node group** and then **Commit 1 change**. 

### Step 4: Run Puppet to pull patch information

1. Run Puppet on your node group by clicking **Run > Puppet** then **Run job**. 

    On this run, Puppet will gather patch information for each of your nodes and store it in the **pe_patch** fact. If the puppet run was successful and with intentional changes, you should be ready to review patch information.


### Step 5: Review high level patch information

1. Click on the **Patches** page, located in the PE sidebar. 

   Here you can see **Nodes under patch management** and actively reporting patch information as well as **Nodes with patches available**.

   #### Patch Filters

   | Filter Dropdowns                 | Function                                                         |
   | -----------                      | -----------                                                      |
   | All patch updates                | Security specific patch updates or regular system/package updates (if the target operating system supports security metadata) |
   | All operating systems            | Filter by a given operating system such as Linux or Windows      |
   | All Patch Groups                 | Choose a specific pre-defined patch group for targeted patching  | 


### Step 6: Review granular patch information

You can review more granular information about a given nodes available patches within the **pe_patch** fact:

1. Navigating to **Nodes** located in the PE sidebar. 
2. Click on your node.
3. Within the **Facts** tab, navigate to the **pe_patch** fact.

    In the **pe_patch** fact you’ll find information on the individual package updates availables, including any security patches that are available (if supported by your OS). You’ll also find information around whether any apps need to be restarted or if a system reboot is required and more.
