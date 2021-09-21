# Overview

Now you understand how roles and profiles are structured, it’s time to create a role for yourself that’ll be made up of multiple profiles. In this lab, you'll create a role that will configure either a basic Apache or IIS webserver depending on your target platform, along with the relevant firewall rules to allow access via HTTP/HTTPS.

You'll perform the following steps:

* Add required modules 
* Create two profiles
* Create a role containing both profiles
* Add profiles and role to the Control Repo
* Review profiles and role content

> This lab assumes you’ve configured SSH and Git on your workstation appropriately and that you’re working from a local copy of the Control Repo within VSCode and pushing to the Control Repo remotely. If you haven’t done this, you can find information on how to do so <a href="https://puppet-enterprise-guide.com/theory/workstation-setup.html" target="_blank">here</a>.

# Steps

### Step 1: Add required modules

> You can skip this step if you’ve previously completed the <a href="https://puppet-enterprise-guide.com/labs/lab-set-up-the-control-repo.html" target="_blank">LAB: Set up the Control Repo</a>. 

1. Navigate to your Control Repo in the Gitlab web interface and add the following modules to your **Puppetfile:**

    #### Linux:

    ```puppet
    mod 'puppetlabs-firewall',                    :latest
    mod 'puppetlabs-apache',                      :latest
    mod 'puppetlabs-stdlib',                      :latest
    ```

    #### Windows:

    ```puppet
    mod 'puppetlabs-stdlib',                      :latest
    mod 'puppetlabs-pwshlib',                     :latest
    mod 'puppetlabs-registry',                    :latest
    mod 'puppetlabs-iis',                         :latest
    mod 'puppet-windows_firewall',                :latest
    ```


### Step 2: Create two profiles

1. Navigate to the **site-modules/profile/manifests** directory.
2. Create the first profile manifest with the following file name and content, relevant to your target platform:

    #### Linux: `lin_firewall.pp`

    ```puppet
    # Profile to open ports for HTTP/HTTPS access
    class profile::lin_firewall {

      firewall { '100 allow http and https access':
        dport  => [80, 443],
        proto  => 'tcp',
        action => 'accept',
      } 
    }
    ```

    #### Windows: `win_firewall.pp`

    ```puppet
    # Profile to open ports for HTTP/HTTPS access
    class profile::win_firewall {

      class { 'windows_firewall':
        ensure => 'running',
      }

      windows_firewall::exception { 'HTTP/HTTPS':
        ensure       => present,
        direction    => 'in',
        action       => 'allow',
        enabled      => true,
        protocol     => 'TCP',
        local_port   => '80,443',
        remote_port  => 'any',
        display_name => 'IIS Webserver access HTTP/HTTPS',
        description  => 'Inbound rule for an IIS Webserver via HTTP/HTTPS [TCP 80,443]',
      }
    }
    ```



3. Save the file.
4. Create the second profile, in the same location **site-modules/profiles/manifests** with the following file name and content, relevant to your target platform:

    #### Linux: `apache.pp`

    ```puppet
    # Profile to install a basic apache webserver
    class profile::apache {
  	  class { 'apache':}
    }
    ```

    #### Windows: `iis.pp`

    ```puppet
    # Profile to install a basic IIS webserver
    class profile::iis {
      $iis_features = ['Web-WebServer','Web-Scripting-Tools']

 	    iis_feature { $iis_features:
   	    ensure => 'present',
 	    }
    }
    ```

5. Save the file.

### Step 3: Create a role containing both profiles

1. Within the **site-modules/role/manifests** directory, create a role with the following file name and content, relevant to your target platform:

    #### Linux: `lin_webserver.pp`

    ```puppet
    class role::lin_webserver {
      include profile::lin_firewall
      include profile::apache
    }
    ```

    #### Windows: `win_webserver.pp`

    ```puppet
    class role::win_webserver {
      include profile::win_firewall
      include profile::iis
    }
    ```

2. Save the file.

### Step 4: Add profiles and role to the Control Repo

1. Ensure both profiles and your role are saved.
2. Commit and push all of your changes to your Control Repo in source control.

### Step 5: Review profiles and role content

1. Review both profiles and role relevant for your platform.

    #### Linux

    In `profile::apache` you’re simply declaring the `apache` class. This is the bare minimum needed to install apache and stand up a webserver. It will leverage the <a href="https://forge.puppet.com/modules/puppetlabs/apache" target="_blank">puppetlabs/apache</a> module to install all the necessary prerequisites and software. It’ll also enable relevant services to ensure the webserver is up and running. 

    `profile::lin_firewall` opens ports 80 and 443 by leveraging the <a href="https://forge.puppet.com/modules/puppetlabs/firewall" target="_blank">puppetlabs/firewall</a> module. As with the apache module, it will also ensure the relevant software is installed and associated services are enabled and running in order to enforce the specified firewall rules.

    #### Windows

    Within `profile::iis `you leverage the <a href="https://forge.puppet.com/modules/puppetlabs/iis" target="_blank">puppetlabs/iis</a> module to enable two features associated with IIS. This is all you need to stand up a really basic IIS webserver. 

    `profile::win_firewall `opens ports 80 and 443 by leveraging the <a href="https://forge.puppet.com/modules/puppet/windows_firewall" target="_blank">puppet/windows_firewall</a> module. As you can see, you declare the **windows_firewall** class and specify `ensure => 'running'`. This will ensure that the windows firewall service is enabled and running in order to enforce the firewall rules specified below within the manifest.