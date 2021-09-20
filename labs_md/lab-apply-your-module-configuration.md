# Overview

So far, you’ve downloaded a module from the Puppet Forge, created a configuration and deployed it to the Puppet Server. In this lab, you'll apply your module configuration on your lab node group and run Puppet to enforce the configuration.

You'll perform the following steps:

* Assign configuration to the lab node group
* Run Puppet to apply & enforce configuration
* Review Puppet run information

# Steps

### Step 1: Assign Configuration to lab node group

1. Ensure your code from previous labs has been deployed.
2. In the Puppet Enterprise web interface, navigate to **Node groups**.
3. Click **Lab Group Linux** _or_ **Lab Group Windows** (whichever is relevant to you)
4. Navigate to the **Classes** tab.
5. On the right hand side, look for “Class definitions updated:” and click the **Refresh** link beneath it.
6. Wait for the text above the link to change to: “Class definitions updated: a few seconds ago”.
7. In the **Add new class** field, type **profile::motd**, click on this profile, then click **Add new class**.
8. On the bottom right hand corner of the page, click **Commit 1 change**.

### Step 2: Run Puppet to apply & enforce configuration

1. On the top right hand corner of the page, click the blue **Run** button, then click **Puppet**.
2. On the page that opens, on the bottom right hand corner of the page, click **Run job**.

### Step 3: Review Puppet run information

1. Under Node run results, you should see that your lab node has **Intentional changes**.
2. To dive in further to what changes took place on that run, click on the time/date stamped report in the **Report** section.
3. Click the dropdown to right of **Filter by event status**, choose **Intentional change**

    #### Linux 

    On Linux, the message content exists as a file. You should see 1 intentional change. This should be a change on a File resource: **/etc/motd**. You’ll notice that under **changed from** / **changed to** the actual content of the file will not be displayed. This is because showing the file content could become very unwieldy as the UI has limited space. Instead, Puppet shows a MD5 hash of the content to prove it was changed - you can decrypt the MD5 hash you’ll see the content of the MOTD file. You can also find the content of the MOTD within in the **Log** tab for this Puppet run.


    #### Windows 

    On Windows, the MOTD is set via registry values. You should see two intentional changes. Both resource changes will be on registry values: **legalnoticecaption** & **legalnoticetext**. Registry values are generally small enough to display in the console however due to the nature of the registry entries you’re working with in this example, the content is too large to be legible from this view. You can find the content of the MOTD within in the **Log** tab for this Puppet run.


4. Navigate to the **Log** tab to find more detailed information on the various actions that took place during the puppet agent run. 

### Step 4: Verify Configuration

1. Verify that the configuration has been created as expected:

    #### Linux 

    Log in to your node via SSH and you should be presented with your MOTD. 

    #### Windows 

    Use RDP to log in into your node and you should see your MOTD appear on the login screen.
