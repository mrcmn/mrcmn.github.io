# Overview

In this lab you’ll run your Plan that you created in the previous lab against your test node in PE.

You'll perform the following steps:

* Inspect Plan information
* Run your Plan
* Review Plan run information

# Steps 

### Step 1: Inspect Plan information


1. From the navigation bar on the left, click on **Plans**.
2. Click **Run a plan** button.
3. Under **Plan** click the dropdown _Enter a plan to run_ and select **adhoc::all_info_linux** or **adhoc::all_info_windows** (whichever is relevant to you) and then click on your Plan.
4. Click **view plan metadata**. You can now see the description as well as available parameters for this plan.

### Step 2: Run Plan

1. For the **Value** of the **myparam1** parameter enter a string, such as “Hello world!”
2. For the **Value** of the **targets** parameter enter your lab node/cert name, for example **node1.company.domain**

    > You can find this information by navigating to **Nodes** and noting the node name.

3. Click **Run job**

### Step 3: Review Plan run information

You should see that the Plan run has succeeded and both steps in the event log have succeeded.


You can click the name of each step to show the output from each task within the Plan.


At the top of the page you’ll see some information relating to the Plan run:


* **Code environment:** This environment in Puppet corresponds to branches of the same name containing code specific to that environment (for example “production” or “development”).
* **Plan name:** Name of Plan chosen for this run
* **Start time:** What time the Plan run was initiated
* **Duration:** How long it took for the run across all target nodes to complete
* **User:** Name of the user that has run the Plan
* **Details:** Inventory details or parameters passed to Plan

In addition to the event log, this information can be used to better understand what has happened when reviewing historical Plan runs, particularly if they’ve been executed by other team members. This Plan is very basic but this information will become more useful when running more complex Plans with many parameters.
