# Overview

Now you understand the basics of all the components that make up Puppet configurations, it’s time to create a manifest with some desired state Puppet code to form your first configuration. In this lab you’ll create a manifest that will create a simple text file with the content “hello world”.

You'll perform the following steps:

* Create a desired state Puppet manifest
* Review manifest content

# Steps

### Step 1: Create a desired state Puppet manifest

1. Navigate to the **manifests** folder - **site-modules/profile/manifests**
2. Click the **+** icon in the navigation bar at the top and select **New file**
3. Name the manifest and add the content according to your platform:

    #### Linux: `lin_file.pp`

    ```puppet
    # Profile to create hello world text file
    class profile::lin_file {
      file { '/tmp/helloworld.txt':
        ensure  => file,
        content => "hello world\n",
      }
    }
    ```

    #### Windows: `win_file.pp`

    ```puppet
    # Profile to create hello world text file
    class profile::win_file {
      file { 'C:\helloworld.txt':
        ensure  => file,
        content => "hello world\n",
      }
    }
    ```

4. Save the file to your Control Repo.

### Step 2: Review manifest content

1. Review the relevant manifest for your platform:

    You’ll see you’re using Puppets built-in <a href="https://puppet.com/docs/puppet/latest/types/file.html" target="_blank">file resource</a> and you’re specifying the file path as the resource title and using `ensure => file` to create the file at this location. You’re also specifying the content within a text file, `helloworld.txt` with the content parameter. Once this configuration is applied, it should result in a file named `helloworld.txt` with the content “hello world” as the first line of the text file.