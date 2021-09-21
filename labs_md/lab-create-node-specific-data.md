# Overview

In the previous lab you added data to the common data layer. This is great for generic values that apply to most nodes but in many cases you want to specify values specific to a given node. 

In this lab you'll add data to the nodes data layer within Hiera and then run Puppet to apply the configuration. 

You'll perform the following steps:

* Create node specific Hiera value
* Review the contents of your node specific YAML file
* Run Puppet to apply and enforce configuration
* Review Puppet run information
* Try out your changes

# Steps

### Step 1: Create node specific Hiera value


1. Within the Control Repo, create a YAML file within the **data/nodes** directory and name it with the <span style="text-decoration:underline;">exact name of your certname</span> (name of your node within the PE console) for example:

    If your node's name in the PE console is **vm.companyname.domain**, then you should name your YAML file `vm.companyname.domain.yaml`.

    You should then have a data directory structure like so: 

    **data/nodes/vm.companyname.domain.yaml** 

    ```
    control-repo/
    └─ data/
        └─ nodes/
            └─ vm.companyname.domain.yaml
    ```

2. Add the following content to the file you’ve just created:

    #### Linux: `data/nodes/{yournodecertname}.yaml`

    ```yaml
    profile::lin_webpage::webcontent: Hello World! This page was generated using data from the nodes data layer in Hiera and is specific to only this node! 
    ```

    #### Windows: `data/nodes/{yournodecertname}.yaml`

    ```yaml
    profile::win_webpage::webcontent: Hello World! This page was generated using data from the nodes data layer in Hiera and is specific to only this node! 
    ```

3. Save the file, commit and push to source control.

### Step 2: Review the contents of your node specific YAML file

Notice that you're specifying data for the `$webcontent` value. This now means that you’re specifying data for the same value in both the **common.yaml** and your **{yournodename}.yaml** file. 

Due to the hierarchical nature of Hiera data layers, this means that the value specified in the highest data layer must be accepted first. Based on the configuration within **control-repo/hiera.yaml**, the value within the **nodes** data layer will be accepted over the **common** data layer.

This means that if you apply the same configuration to another node without specifying relevant data in the **nodes** data layer, it will automatically receive the value stored in the  **common** data layer.

### Step 3: Run Puppet to apply and enforce configuration

1. From the sidebar, under the Enforcement section, choose **Jobs**.
2. On the top right hand corner of the page, click the blue **Run Puppet** button.
3. From the **Inventory** drop down, click on **Select a target type** and choose **Node Group**.
4. In the _Enter a node group_ text field, choose **Lab Group**, then click **Select**. You should now see your node.
5. From the bottom right hand corner of the page, click **Run job**.

### Step 4: Review Puppet run information

1. Under Node run result, you should see that your lab node has **Intentional changes**.
2. To dive in further to what changes took place on that run, click on the time/date stamped report in the **Report** section.
3. Click the dropdown to right of **Filter by event status**, choose **Intentional change**:

    #### Linux

    On Linux, you should see 1 intentional change. This should be a change to a file resource: **/var/www/html/index.html**. You’ll notice that under **changed from** / **changed to** the actual content of the file will not be displayed. This is because showing the file content could become very unwieldy as the UI has limited space. Instead, Puppet shows a MD5 hash of the content to prove it was changed. 

    #### Windows

    On Windows, you should see 1 intentional change. This should be a change to a file resource: **C:\Inetpub\wwwroot\index.html**. You’ll notice that under **changed from** / **changed to** the actual content of the file will not be displayed. This is because showing the file content could become very unwieldy as the UI has limited space. Instead, Puppet shows a MD5 hash of the content to prove it was changed. 


### Step 5: Try out your changes

1. Navigate to your node's public IP address or hostname to see your new custom web content.

    Notice that web content has now changed to the value specified in the nodes data layer due to it’s hierarchical precedence over the common data layer value.

    > You can quickly get your public ip address by navigating to **Nodes**, clicking on the name of your node and then from the **facts** tab, you can find your public IP address from the **ipaddress** fact value. You can also find your hostname from the **hostname** fact value.
