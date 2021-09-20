# Overview

In this lab, you’ll inspect your task that you created in the previous lab through Task Metadata and then run it against your lab node in Puppet Enterprise.

You'll perform the following steps:


* Inspect task information via Task metadata
* Run your Task 
* Review Task run output

# Steps

### Step 1: Inspect task information via Task metadata



1. In the Puppet console under Orchestration, choose **Tasks**
2. Click **Run a task** button
3. Under **Task** click the dropdown _Enter a task to run_ and type **adhoc::lin_info** _or_ **adhoc::win_info** (whichever is relevant to you) and then click on your Task
4. Click **view task metadata**

    You can see the description and parameters for your task now visualised within the PE console. This is “output” of the information specified within the task metadata file, created in the previous lab. 


### Step 2: Run your Task

1. Enter some random text into the **myparam1** parameter under **Task parameters**
2. Scroll down the page and click the dropdown **Select a target type**
3. Choose **Node group** and then select or type the Node group name **Lab Group Linux** _or_ **Lab Group Windows** (whichever is relevant to you) then Click **Select**
4. Click **Run task**

### Step 3: Review Task output


Once your Task has successfully completed, your task output will be displayed at the bottom of the page. 


At the top of the page you’ll see some information relating to the task run:



* **Code environment:** This environment in puppet corresponds to branches of the same name containing code specific to that environment (for example “production” or “development”).
* **Task name:** Name of Task chosen for this run
* **Start time:** What time the Task run was initiated
* **Duration:** How long it took for the run across all target nodes to complete
* **User:** Name of the user that has run the Task
* **Details:** Inventory details or parameters passed to Task

This information can be used to better understand what has happened when reviewing historical task runs, particularly if they’ve been executed by other team members. 

