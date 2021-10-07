# Overview

In this lab, you'll apply any patches available to your node using the built in `pe_patch::patch_server` task.

You'll perform the following steps:

* Run the pe_patch::patch_server task
* Review high level patch information
* Review granular patch information

> If your lab node doesn’t have any patches available, you will not be able to complete this lab.

# Steps

### Step 1: Run the pe_patch::patch_server task

1. Navigate to the **Patches** page, located in the PE sidebar 
2. From the **All Patch Groups** dropdown, select **Patch Group 1**
3. Click **Run** then **Task**
4. Ensure the reboot parameter is set to **patched** (this will reboot node after patches are applied)
5. Click **Run task** - Once the task run completes successfully, proceed to step 2.

### Step 2: Review high level patch information

1. Navigate back to the **Patches** page.

   You should find that your node is no longer there as patches have now been applied. 

### Step 3: Review granular patch information

1. Navigate to **Nodes** located in the PE sidebar.
2. Click on your node.
3. Within the **Facts** tab navigate to the **pe_patch** fact.

   Within the **pe_patch** fact you’ll find information on the individual package updates availables, including any security patches that are available (if supported by your OS). You’ll also find information around whether any apps need to be restarted or if a system reboot is required and more.
