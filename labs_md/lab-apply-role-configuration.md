# Overview

In this lab you'll apply your role configuration to your lab node group and then run puppet to apply and enforce the configuration.

You'll perform the following steps:

* Assign Configuration to Lab node group
* Run Puppet to apply & enforce configuration
* Review Puppet run information
* Try out your changes

# Steps

### Step 1: Assign Configuration to Lab node group

1. In the Puppet Enterprise web interface, navigate to **Node groups**
2. Click **Lab Group Linux** _or_ **Lab Group Windows** (whichever is relevant to you)
3. Navigate to the **Classes tab**.
4. On the right hand side, look for “Class definitions updated:” and click the **Refresh** link beneath it
5. Wait for the text above the link to change to: “Class definitions updated: a few seconds ago”
6. In the **Add new class** field, type **role::lin_webserver** _or_ **role::win_webserver** (whichever is relevant to you), click on this profile, then click **Add new class**.
7. In the bottom right hand corner of the page, click **Commit 1 change**.

### Step 2: Run Puppet to apply & enforce configuration

1. On the top right hand corner of the page, click the blue **Run** button, then click **Puppet**.
2. On the page that opens, on the bottom right hand corner of the page, click **Run job**

### Step 3: Review Puppet run information

1. Under Node run result, you should see that your lab node has **intentional changes**.
2. To dive in further to what changes took place on that run, click on the time/date stamped report in the **Report** section.
3. Click the dropdown to right of **Filter by event status**, choose **Intentional change**

    #### Linux

    You should see several **intentional changes** on multiple resources. All of which are the necessary components to stand up the apache software and associated service. Depending on your current firewall settings, you may or may not see changes related firewall ports. If your current state already matched the desired state of the firewall profile, then puppet will not make any changes.

    #### Windows

    You should see a few **intentional changes** on multiple resources. You should see some changes related to the IIS features enabled in the iis profile. You will also see changes to  firewall rules in alignment with the firewall profile.


### Step 4: Try out your changes

1. Navigate to your node's public IP address or hostname to view default content from your webserver

    > You can quickly get your public ip address by navigating to **Nodes**, clicking on the name of your node and then from the **facts** tab, you can find your public IP address from the **ipaddress** fact value. You can also find your hostname from the **hostname** fact value.