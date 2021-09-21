# Overview

In the previous lab, you created a role consisting of 3 different profiles, however all of your parameter values were mixed with Puppet code. By using Hiera variables, you can easily separate data from code. When Puppet manifests with Hiera variables are reconciled, Puppet will automatically look for the data of those variables in Hiera. 

In this lab, you’ll remove a hard coded value and make it a dynamic variable that can be populated via Hiera.

You'll perform the following steps:

* Create a Hiera variable
* Review updated manifest

# Prerequisites

> You will need to have completed the Roles and Profiles labs <a href="https://puppet-enterprise-guide.com/labs/lab-build-a-role.html" target="_blank">1</a> + <a href="https://puppet-enterprise-guide.com/labs/lab-apply-role-configuration.html" target="_blank">2</a> and <a href="https://puppet-enterprise-guide.com/labs/lab-add-to-your-role.html" target="_blank">3</a> prior to working with this lab.

# Steps

### Step 1: Create a Hiera variable

1. Navigate to your Control Repo and open your web page manifest from and edit the file to accommodate a dynamic Hiera value, as shown below: 

    #### Linux: `site-modules/profiles/manifests/lin_webpage.pp`

    ```puppet
    # Profile to add custom webserver content
    class profile::lin_webpage (
      String $webcontent
    ) {

      file { '/var/www/html/index.html':
        ensure  => file,
        content => $webcontent,
      }
    }
    ```

    #### Windows: `site-modules/profiles/manifests/win_webpage.pp`

    ```puppet
    # Profile to add custom webserver content  
    class profile::win_webpage (
      String $webcontent
    ) {

      file { 'C:\Inetpub\wwwroot\index.html':
        ensure  => file,
        content => $webcontent,
      }
    }
    ```



2. Save the file - don’t commit and push changes just yet.

### Step 2: Review updated manifest

Review the contents of `profile::lin_webpage` _or_ `profile::win_webpage `(whichever is relevant to you)

Notice that the value of “content” is now set to a dynamic variable `$webcontent`, which is now declared at the top of the class in parentheses. This variable can be automatically populated via Hiera. 

In this next lab, you’ll now add your desired web content data in Hiera which Puppet will automatically pull and populate as part of this profile when a Puppet agent run occurs on your node. 
