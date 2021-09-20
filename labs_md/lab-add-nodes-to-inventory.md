# Overview

In this lab, you'll begin to build inventory by connecting to your target node and deploying a Puppet agent to allow Puppet server to orchestrate change and enforce state on this node.

You'll perform the following steps:

* Install Puppet agent
* Approve agent certificate
* Perform Puppet run
* Verify Inventory

> If your target node is running a Linux operating system which is different from the puppet server platform, i.e. puppet server (CentOS) and target node (Ubuntu) you will need to download the relevant puppet agent packages to the puppet server. Review the <a href="https://puppet-enterprise-guide.com/theory/node-inventory.html#download-agent-packages" target="_blank">Download agent packages</a> section of the Node Inventory page in this guide. Alternatively, you can review the <a href="https://puppet.com/docs/pe/latest/installing_agents.html" target="_blank">relevant puppet docs</a>.

# Steps

### Step 1: Install Puppet agent

1. From the navigation bar on the left, click on **Nodes**.
2. Click **Add nodes** and then **Install Agents**.
3. **Transport Method:**
   

    #### SSH (default)

    1. Add the hostname or IP address of your target node in the **SSH hosts** field.
    2. Input the **User** to be used on the target node via SSH (this user must be able to run sudo commands without a password).
    3. Choose whether you are authenticating with an **SSH key** or **Password** and input the relevant credential information below.
    4. Ensure that the **Test Connections** option is checked and then click **Add nodes**.
    5. Click on the **Installation Job Started** link to view the progress of the installation and confirm that Puppet agent installation was successful.

    #### WinRM

    1. Add the hostname or IP address of your target node in the **WinRM hosts** field.
    2. Input the **User** to be used on the target node via WinRM.
    3. Input the **Password** for the username provided.
    4. Ensure that the **Test Connections** option is checked and then click **Add nodes**.
    5. Click on the **Installation Job Started** link to view the progress of the installation and confirm that Puppet agent installation was successful.

    
    > If you have authentication/connectivity issues with SSH or WinRM, you can manually install the agent by running the relevant commands on your chosen platform shown on the right hand side of the page.

### Step 2: Approve Agent certificate

1. Click on **Certificates**.
2. Click on the **Unsigned certificates** tab.
3. You should see a certificate request with a node name matching your target node - on the right of the certificate request click **Accept**.

> After you accept the certificate signing request, your node may not appear in the console immediately. It will be visible whenever the next puppet run has taken place (default 30 mins). To make a node available immediately after you approve a request, you may need to trigger another puppet run by executing the relevant puppet command on the agent node manually, as described below in Step 3.


In the future, you can configure <a href="https://puppet.com/docs/puppet/latest/config_file_autosign.html" target="_blank">auto-signing</a> to automatically sign certificates on agents from known-good servers or domains to be accepted immediately.


### Step 3: Perform Puppet run



1. Initiate a Puppet run directly on your lab node

    #### Linux

    On the command line run:

    ```
    puppet agent -t
    ```

    #### Windows

    Open Command prompt as an administrator and then run:

    ```
    puppet agent -t
    ```


### Step 4: Verify Inventory



1. In the puppet console, under **Inventory** navigate to **Nodes**. You should now see your node.