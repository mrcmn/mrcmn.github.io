# Overview

Now that you’ve created a user, user role and set it’s permissions, it’s time to test it out. In this lab, you’ll login into the console with your newly created user and experience the limited permissions that you configured in the previous lab.

You'll perform the following steps:

* Login with lab user
* View and edit your node group
* Run a Task
* Run a Plan

# Steps

### Step 1: Login with lab user

1. Logout of the Puppet console and login with your newly created lab user.

### Step 2: View and edit your node group

1. From the navigation bar on the left, click on **Node Groups**

   You’ll notice that you can no longer view other node groups, other than the ones specified in your users role permissions. 

2. Click on your node group, and from the **Rules** tab, try to unpin your node by clicking **Unpin**, to the right of your node name under **Certname**.

   You should see a red banner with an error message at the top of the page explaining that you don’t have sufficient permissions to make this change.


### Step 3: Run a task

1. From the navigation bar on the left, Click on **Tasks**.
2. Click **Run a task** button.
3. Under **Task** click the dropdown _Enter a task to run_ and type **adhoc::lin_info** or **adhoc::win_info** (whichever is relevant to you) and then click on your Task.
4. Scroll down the page and click the dropdown **Select a target type**.
5. Choose **Node group** and then click the dropdown _Enter a node group_ and choose **Lab Group Linux** or **Lab Group Windows** then Click **Select**
6. Click **Run task**

   Notice that the available options for Tasks and node groups are now limited to the ones specified in your lab user permissions.


### Step 4: Run a Plan

1. From the navigation bar on the left, click on **Plans**
2. Click **Run a plan** button
3. Under **Plan** click the dropdown _Enter a plan to run_ and select **adhoc::all_info_linux** or **adhoc::all_info_windows** (whichever is relevant to you) and then click on your Plan.

   You’ll see that you only have one option when choosing a plan, inline with lab users permissions

4. For the **Value** of the **myparam1** parameter, enter a custom string, such as “Hello world!”
5. For the **Value** of the **targets** parameter enter your lab node name, for example **node1.company.domain**

   > You can find this information by clicking on **Nodes** from the navigation bar, and noting the node name.

6. Click **Run job**