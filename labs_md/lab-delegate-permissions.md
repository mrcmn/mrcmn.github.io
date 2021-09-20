# Overview

By leveraging Role based access control (RBAC) in Puppet Enterprise, you can easily manage permissions to enable everyone to use Puppet Enterprise in a safe and controlled manner. In this lab you'll create a new user and delegate permissions to limit the actions the user can take within PE.

You'll perform the following steps:

* Create a new user
* Create a user Role
* Add the user to the user role
* Set permissions for the user role

> This lab expects that you to have completed the following labs: Task labs <a href="https://puppet-enterprise-guide.com/labs/lab-create-a-task.html" target="_blank">1</a> + <a href="https://puppet-enterprise-guide.com/labs/lab-run-your-task.html" target="_blank">2</a>, Plan labs <a href="https://puppet-enterprise-guide.com/labs/lab-create-a-plan.html" target="_blank">1</a> + <a href="https://puppet-enterprise-guide.com/labs/lab-run-your-plan.html" target="_blank">2</a>.

# Steps

### Step 1: Create a new User

1. In the Puppet Enterprise web interface, navigate to **Access Control** from in the navigation bar on the left.
2. Add a new local user by filling in the fields on the top line and clicking the **Add local user** button:
    1. Full name: **_Lab User_**
    2. Login: **_lab-user_**
3. Click on the name of your newly created user.
4. Click **Generate password reset**.
5. Copy **Password reset link** in a new browser window/tab and set a password for your new user. Make a note of this username & password.

### Step 2: Create a User Role

1. Navigate back to **Access control** and click the **User roles** tab.
2. Under **Name** enter **"Lab Test"** and then click **Add Role**.

### Step 3: Add User to User role

1. Click on your newly created role - **Lab Test**.
2. Under **User name** choose your newly created Lab User and then click **Add user**.
3. Click **Commit 1 change**.

### Step 4: Set permissions for User Role

1. Navigate to the **Permissions** tab.
2. Under **Add a permission**, add the following permissions:

    #### Linux

    * Console - **View**
    * Job orchestrator - **Start, stop and view jobs**
    * Node Groups - View - **Lab Group Linux (production)**
    * Tasks - Run Tasks - **adhoc::lin_info**

        Permitted Nodes - Node Group - **Lab Group Linux (production)** 

    * Tasks - Run Tasks - **adhoc::lin_network**

        Permitted Nodes - Node Group - **Lab Group Linux (production)** 

    * Plans - Run Plans - **adhoc::all_info_linux**

    #### Windows

    * Console - **View**
    * Job orchestrator - **Start, stop and view jobs**
    * Node Groups - View -** Lab Group Windows (production)**
    * Tasks - Run Tasks - **adhoc::win_info**

        Permitted Nodes - Node Group - **Lab Group Windows (production)**

    * Tasks - Run Tasks - **adhoc::win_network**

        Permitted Nodes - Node Group - **Lab Group Windows (production)**

    * Plans - Run Plans - **adhoc::all_info_windows**

3. When all permissions are configured, click **Commit 6 changes**.