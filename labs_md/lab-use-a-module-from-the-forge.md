# Overview

Now you know what the Puppet Forge is as well as how and why you use it, you’re going to put theory into practice.

In this lab you'll configure your lab node with a message of the day (MOTD) banner. To do this, you’ll leverage the MOTD module from Puppet Forge. In your Puppet manifest you’ll dynamically populate some key system information in the MOTD message via Facter. 

**Facter**

Facter is a cross platform system profiling tool that automatically gathers key system information and stores them as “facts” in Puppet DB on the Puppet server. Facter is an integral part of the Puppet agent and therefore means this process happens automatically each time a Puppet agent run is performed.

You'll perform the following steps:

* Download the MOTD module from the Forge 
* Create MOTD Puppet manifest
* Review manifest content

# Steps

### Step 1: Download MOTD Module from the Forge

When downloading modules from the Puppet Forge, you must **always ensure** that you not only add the module to your Puppetfile but also any relevant dependencies. You can find this information on the <a href="https://forge.puppet.com/puppetlabs/motd/dependencies" target="_blank">dependencies</a> tab on the module listing on the Forge.


The MOTD module has several dependencies however you’ve have already added them as part of your initial Puppetfile back in Step 8 of <a href="https://puppet-enterprise-guide.com/labs/lab-set-up-the-control-repo.html" target="_blank">LAB: Set up the Control Repo</a>, so in this case you’ll only need to focus on adding the MOTD module itself.


1. Within the Control Repo in the Gitlab web interface, navigate to the **Puppetfile** in the root of the control repo and add this line:

    ```puppet
    mod 'puppetlabs-motd',      :latest
    ```
    _or_ if you haven't completed Step 8 of <a href="https://puppet-enterprise-guide.com/labs/lab-set-up-the-control-repo.html" target="_blank">LAB: Set up the Control Repo</a>:

    ```puppet
    mod 'puppetlabs-motd',      :latest
    mod 'puppetlabs-registry',  :latest
    mod 'puppetlabs-stdlib',    :latest
    ```

2. Save the file.

### Step 2: Create MOTD Puppet manifest

The MOTD module is fairly unique in that it is a cross platform module and employs an abstraction layer across both Windows and Linux. This means that you can use the same basic Puppet code, so long as parameters used are generic across both Linux and Windows. This is why the manifest below will work with both Linux and Windows nodes.

Whilst a basic level of cross platform abstraction has it’s advantages, more advanced configuration usually requires platform specific parameters. Puppet replicates these parameters in Puppet code to ensure that Linux and Windows practitioners can easily translate existing platform configuration expertise into Puppet configurations. 

This means that if you want to take advantage of some of the <a href="https://forge.puppet.com/modules/puppetlabs/motd/reference#classes-1" target="_blank">OS-specific options in this module</a>, you will need to create a seperate configuration for each platform. For lab purposes, you'll create a fairly basic example configuration that will work across both platforms.


1. Navigate to the **site-modules/profile/manifests** directory and create a manifest with the file name and content shown below:

    #### Linux/Windows: `motd.pp`

    ```puppet
    # Profile to configure MOTD
    class profile::motd {

    # Puppet Heredoc variable containing MOTD content
    $message = @("MOTD"/L)
        = = = = = = = = = = = = = = = = = = = = = = = = = = =
        |                                                    
        |   Welcome to ${::hostname}                                              
        |                                                     
        |   Access to and use of this server is restricted to    
        |   those activities expressly permitted by            
        |   the system administration staff.                   
        |                                                    
        |   If you are not sure if it is allowed then          
        |   DO NOT DO IT.    
        |
        = = = = = = = = = = = = = = = = = = = = = = = = = = =
        |
        |   Domain: ${::domain}
        |   Operating System: ${::operatingsystem} ${::operatingsystemmajrelease}
        |   
        = = = = = = = = = = = = = = = = = = = = = = = = = = =

        | MOTD

    # Using MOTD module to display $message content as MOTD
      class { 'motd':
        content => $message,
      }
    }
    ```

2. Save the file.

### Step 3: Review manifest content

For the sake of simplicity, the MOTD message is defined as a rather large variable `$message` at the top of Puppet code manifest, which is then passed below to the module via the `motd` class as the desired message content. 

In the Going Further labs, you’ll learn how to store content like this in template files and refer to them in Puppet code, allowing you to keep your code neat, maintainable and easy to understand.

Notice the areas where the manifest is leveraging Facter for key system information such as **hostname**, **domain** **operating system** and **operating system major release**.