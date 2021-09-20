# Overview

In this lab, you'll apply your newly created configuration on your target node. Once you've applied your configuration, Puppet will continually enforce the desired state of this configuration.

You'll perform the following steps:

* Assign configuration to the lab node group
* Run Puppet to apply & enforce configuration
* Review Puppet run information
* Verify Configuration
* Delete Configuration and perform a Puppet run

# Steps

### Step 1: Assign configuration to lab node group

1. Ensure your code from previous labs has been deployed.
2. In the Puppet Enterprise web interface, navigate to **Node groups**.
3. Click **Lab Group Linux** _or_ **Lab Group Windows** (whichever is relevant to you).
4. Navigate to the **Classes** tab.
5. On the right hand side, look for “Class definitions updated:” and click the **Refresh** link beneath it.
6. Wait for the text above the link to change to: “Class definitions updated: a few seconds ago”.
7. In the **Add new class** field, type **profile::lin_file** _or_ **profile::win_file** (whichever is relevant to you), click on your profile, then click **Add new class**.
8. In the bottom right hand corner of the page, click **Commit 1 change**.

### Step 2: Run Puppet to apply & enforce configuration

1. On the top right hand corner of the page, click the blue **Run** button, then click **Puppet**.
2. On the page that opens, on the bottom right hand corner of the page, click **Run job**.

### Step 3: Review Puppet run information

1. Under Node run result, you should see that your lab node has **intentional changes**.
2. To dive in further to what changes took place on that run, click on the time/date stamped report in the **Report** section.
3. Click the dropdown to right of **Filter by event status**, choose **Intentional change**:

    #### Linux 

    You should see **1 intentional change**. This should be a change on a File resource: `/tmp/helloworld.txt`. If you click on the file resource, you’ll see the exact Puppet configuration file and line number where this file resource is specified. You’ll notice that under **changed from** / **changed to** the actual content of the file will not be displayed. This is because showing the file content could become very unwieldy as the UI has limited space. Instead Puppet shows a MD5 hash of the content to prove it was changed - found in the **Log** tab. If you decrypt the MD5 hash you’ll see the content “hello world”.

    #### Windows 

    You should see **1 intentional change**. This should be a change on a File resource: `C:\helloworld.txt`. If you click on the file resource, you’ll see the exact Puppet configuration file and line number where this file resource is specified. You’ll notice that under **changed from** / **changed to** the actual content of the file will not be displayed. This is because showing the file content could become very unwieldy as the UI has limited space. Instead Puppet shows a MD5 hash of the content to prove it was changed (found in the **Log** tab). If you decrypt the MD5 hash you’ll see the content “hello world".

4. Navigate to the **Log** tab to find more detailed information on the various actions that took place during the puppet agent run. 

### Step 4: Verify Configuration

1. Verify that the configuration has been created as expected:

    #### Linux 

    Log in to your node via SSH and run `cat /tmp/helloworld.txt`. 

    #### Windows 

    Use RDP to log in into your node and navigate to your `C:\` drive and open up the `helloworld.txt` file.

### Step 5: Delete Configuration and perform a puppet run

Not only has Puppet applied this configuration but it is now continually enforcing it. This means that if you delete this file, Puppet will put this file back whenever your node checks in with the Puppet Server (default is 30 minutes). To simulate this, we can delete the file and run Puppet immediately after to see the result.

1. Delete the `helloworld.txt` file from your node.
2. In the Puppet console, navigate to **Node Groups** then click on **Lab Group Linux** _or_ **Lab Group Windows** (whichever is relevant to you).
3. In the top right of the page, click the blue **Run** button, then click **Puppet**.
4. Now the puppet run page has appeared, On the bottom right hand corner of the page, click **Run job**.

You should now see 1 **Corrective change**. If you click on the time/date stamped report in the **Report** section, you can then click **Filter by event status** and choose **Corrective change**. You'll now see a file resource change on `helloworld.txt`.
