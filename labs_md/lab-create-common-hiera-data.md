# Overview

In the previous lab, you created a Hiera variable within a manifest to enable the content of your file to be dynamically populated via Hiera, however you still need to specify tge corresponding data within a Hiera data layer. 

In this lab, you'll add your data to the common data layer within Hiera and then run Puppet to apply the configuration. 

You'll perform the following steps:

* Create a value in the common data layer
* Review common data content
* Run Puppet to apply and enforce configuration
* Review Puppet run information
* Try out your changes

# Steps

### Step 1: Create a value in the common data layer

1. Within the Control Repo, open the common.yaml from the data directory and add the content relevant to your platform:

    #### Linux: `data/common.yaml`

    ```yaml
    profile::lin_webpage::webcontent: Hello World! This page was generated using data from the common data layer in Hiera!
    ```

    #### Windows: `data/common.yaml`

    ```yaml
    profile::win_webpage::webcontent: Hello World! This page was generated using data from the common data layer in Hiera!
    ```

2. Save the file, commit 2 changes and push to source control.

### Step 2: Review common data content

Review the contents of the `common.yaml` file.

There are two key things you need to do in order to create a Hiera variable in a data layer:

1. Specify the location of the Hiera variable you want to populate within a manifest.
2. Specify the variable value itself.

    The syntax follows the general Puppet naming convention but in this instance you specify the Hiera variable name along with it’s value. The syntax is as follows: 

    **CLASSNAME::HIERAVARNAME: VARIABLEVALUE**

    Once Puppet has reconciled this Hiera data value during a Puppet run, you should expect to see that it appears on the webpage hosted on your vm’s web server.

    > When specifying data in the common layer, all nodes will receive this value unless the same parameter is specified in a data layer that is higher in the hierarchy specified in the `hiera.yaml` file.

### Step 3: Run Puppet to apply and enforce configuration

1. From the sidebar, under the Enforcement section, choose **Jobs**.
2. On the top right hand corner of the page, click the blue **Run Puppet** button.
3. From the **Inventory** drop down, click on **Select a target type** and choose **Node Group**.
4. In the _Enter a node group_ text field, choose **Lab Group Linux** _or_ **Lab Group Windows** (whichever is relevant to you), then click **Select**. You should now see your node displayed below.
5. From the bottom right hand corner of the page, click **Run job**.

### Step 4: Review Puppet run information

1. Under Node run result, you should see that your lab node has **Intentional changes**.
2. To dive in further to what changes took place on that run, click on the time/date stamped report in the **Report** section.
3. Click the dropdown to right of **Filter by event status**, choose **Intentional change**:

    #### Linux:

    On Linux, you should see 1 intentional change. This should be a change to a file resource: **/var/www/html/index.html**. You’ll notice that under **changed from** / **changed to** the actual content of the file will not be displayed. This is because showing the file content could become very unwieldy as the UI has limited space. Instead, Puppet shows a MD5 hash of the content to prove it was changed. 

    #### Windows:

    On Windows, you should see 1 intentional change. This should be a change to a file resource: **C:\Inetpub\wwwroot\index.html**. You’ll notice that under **changed from** / **changed to** the actual content of the file will not be displayed. This is because showing the file content could become very unwieldy as the UI has limited space. Instead, Puppet shows a MD5 hash of the content to prove it was changed. 


### Step 5: Try out your changes

1. Navigate to your node's public IP address or hostname to see your new custom web content generated from the Hiera common data layer.

    > You can quickly get your public ip address by navigating to **Nodes**, clicking on the name of your node and then from the **facts** tab, you can find your public IP address from the **ipaddress** fact value. You can also find your hostname from the **hostname** fact value.
