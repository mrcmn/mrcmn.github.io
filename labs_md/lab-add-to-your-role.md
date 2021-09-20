# Overview

In this lab, you'll create a simple profile that will populate the webserver webpage with custom content, replacing the default webpage. You'll then add this profile to your role and run puppet to apply the new configuration.

You'll perform the following steps:

* Create a profile
* Add profile to your role
* Review profile and role content
* Run Puppet to apply and enforce configuration
* Review Puppet run information
* Try out your changes

# Steps

### Step 1: Create a profile

1. Navigate to your Control Repo and under the **site-modules/profiles/manifests** and create a profile with the following file name and content, relevant to your target platform:

    #### Linux: `lin_webpage.pp`

    ```puppet
    # Profile to add custom webserver content
    class profile::lin_webpage {

      file { '/var/www/html/index.html':
        ensure  => file,
        content => "Hello World!",
      }
    }
    ```

    #### Windows: `win_webpage.pp`

    ```puppet
    # Profile to add custom webserver content  
    class profile::win_webpage {

      file { 'C:\Inetpub\wwwroot\index.html':
        ensure  => file,
        content => "Hello World!",
      }
    }
    ```

2. Save the file.

### Step 2: Add profile to your role

1. Open the **site-modules/role/manifests/webserver.pp** file and add your profile declaration to it like so:

    #### Linux:

    ```puppet
    class role::lin_webserver {
      include profile::lin_firewall
      include profile::apache
      include profile::lin_webpage
    }
    ```

    #### Windows:

    ```puppet
    class role::win_webserver {
      include profile::win_firewall
      include profile::iis
      include profile::win_webpage
    }
    ```

2. Save the file then commit your 2 changes and push to source control.

### Step 3: Review profile and role content

1. Review both profiles and role relevant for your platform:

    #### Linux

    In `profile::lin_webpage` you’re creating a new default web page using the file resource. You’re specifying the path specified should be a “file” and it’s content should consist of “Hello World!”. This content will be displayed whenever you apply this configuration and then navigate to the IP address of your target node.

    `role::lin_webserver` has one small change to include the newly created profile as part of the role by specifying include `profile::lin_webpage`.

    #### Windows

    In `profile::win_webpage` you’re creating a new default web page using the file resource. You’re specifying the path specified should be a “file” and it’s content should consist of “Hello World!”. This content will be displayed whenever you apply this configuration and then navigate to the IP address of your target node.

    `role::win_webserver` has one small change to include the newly created profile as part of the role by specifying include `profile::win_webpage`.

### Step 4: Run Puppet to apply and enforce configuration

1. From the sidebar, under the Enforcement section, choose **Jobs**
2. On the top right hand corner of the page, click the blue **Run Puppet** button.
3. From the **Inventory** drop down, click on **Select a target type** and choose **Node Group**.
4. In the _Enter a node group_ text field, choose **Lab Group Linux** _or_ **Lab Group Windows** (whichever is relevant to you), then click **Select**. You should now see your node.
5. From the bottom right hand corner of the page, click **Run job**.

### Step 5: Review Puppet run information

1. Under Node run result, you should see that your lab node has **intentional changes**.
2. To dive in further to what changes took place on that run, click on the time/date stamped report in the **Report** section.
3. Click the dropdown to right of **Filter by event status**, choose **Intentional change**

    #### Linux

    You should see 1 intentional change. This should be a change to a File resource: **/var/www/html/index.html**. You’ll notice that under **changed from** / **changed to** the actual content of the file will not be displayed. This is because showing the file content could become very unwieldy as the UI has limited space. Instead, Puppet shows a MD5 hash of the content to prove it was changed. 

    #### Windows

    You should see 1 intentional change. This should be a change to a File resource: **C:\Inetpub\wwwroot\index.html**. You’ll notice that under **changed from** / **changed to** the actual content of the file will not be displayed. This is because showing the file content could become very unwieldy as the UI has limited space. Instead, Puppet shows a MD5 hash of the content to prove it was changed. 


### Step 6: Try out your changes

1. Navigate to your node's public IP address or hostname to see your custom web content.

    > You can quickly get your public ip address by navigating to **Nodes**, clicking on the name of your node and then from the **facts** tab, you can find your public IP address from the **ipaddress** fact value. You can also find your hostname from the **hostname** fact value.
