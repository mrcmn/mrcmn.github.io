# Overview

In this lab, you will set up and configure a Control Repo in order to store and use configurations within Puppet Enterprise.

You'll perform the following steps:

- Create a project for the Control Repo on your git server
- Clone template Control Repo and push to your git server
- Setup SSH keys on the Puppet Enterprise server
- Set a Deploy Key on the Control Repo
- Enable Code Manager on Puppet Enterprise
- Deploy the Control Repo to Puppet Enterprise
- Enable a webhook for auto sync of your Control Repo updates
- Add some useful modules to your Puppet Enterprise installation

> This lab assumes you’re using Gitlab Community Edition as your git server. If you’re using a different git technology, you’ll have to interpret some of the steps of navigating around the Gitlab interface and translate that to the git technology you’re using.

# Steps


### Step 1: Create a project for the Control Repo on your git server



1. Login to Gitlab web interface with an account that can create new groups and projects.
2. Create a new group called "puppet":
    1. On the top navigation bar, click **Groups** → **Your groups**
    2. Click the green **New group** button on the right
    3. Set the **Group name** to: puppet
    4. Leave the **Visibility level** at its default of Private
    5. Click **Create group**
    6. You’ll land in the group’s overview page
3. Create a new project called "control-repo":
    1. Click the **New project** button on the right
    2. If Gitlab asks what type of new project to create, select **Create blank project**
    3. Set the **Project name** to: control-repo
    4.  Leave the **Visibility level** at its default of Private
    5.  Leave the **Initialize repository with a README** option unchecked
    6.  Click **Create project**


 ### Step 2: Clone template Control Repo and push to your git server

Login to the terminal console of your Puppet Enterprise server with the <span style="text-decoration:underline;">root account</span>. It’s best to do this via SSH, so that you can easily copy & paste the commands from this Lab.

1. First, you need to install the git CLI tools:
   ```shell
   yum install -y git
   ```
2. Setup a git working environment - be sure add _your own_ name and email address in the git config command: 
   ```shell
   cd ~ 
   mkdir code 
   cd code 
   ```
   ```shell
   git config --global user.name "Your Name" 
   ```
   ```shell
   git config --global user.email your.email@address.com 
   ```

3. Clone the template Control Repo locally:

   ```shell
   git clone https://github.com/puppetlabs/control-repo.git --branch production
   cd control-repo
   ```


4. Reconfigure the repo to point it to the git server:

   ```shell
   git remote remove origin
   ```

   ```markdown
   git remote add origin http://<fqdn of your gitlab server>/puppet/control-repo.git
   ```


5. Finally, push the repo to the git server (this command will ask for your credentials to the git server):

   ```shell
   git push origin production
   ```

      You should see output similar to the following: 

   <div class="noninteractive">

   ```markdown
   Counting objects: 858, done.
   Delta compression using up to 4 threads.
   Compressing objects: 100% (400/400), done.
   Writing objects: 100% (858/858), 150.97 KiB | 0 bytes/s, done.
   Total 858 (delta 364), reused 858 (delta 364)
   remote: Resolving deltas: 100% (364/364), done.
   To http://gitlab.company.local/puppet/control-repo.git
   * [new branch]      production -> production
   ```
   </div>

6. If you now refresh the webpage of the project you created in Step 1, you’ll notice the project is now populated with the template Control Repo content.


 ### Step 3: Setup SSH keys on the Puppet Enterprise server


You’ll now create some SSH keys, so that the primary puppet server can use them to gain remote access to the Control Repo you just set up.


1. Generate a fresh pair of SSH keys:

   ```shell
   cd ~
   cat /dev/zero | ssh-keygen -q -N ""
   ```

    The newly generated keypair has now been stored in ~/.ssh

2. Move the private key of this pair in preparation for use with Code Manager

   ```shell
   cp ~/.ssh/id_rsa /etc/puppetlabs/puppetserver/ssh/id-control_repo.rsa
   ```

   ```shell
   chown pe-puppet:pe-puppet /etc/puppetlabs/puppetserver/ssh/id-control_repo.rsa
   ```


 ### Step 4: Set a Deploy Key on the Control Repo

To ensure the primary puppet server has access to the Control Repo on your git server, you’ll need to set the public key to be a Deploy Key on the repo in Gitlab.

1. On the Puppet Enterprise server terminal console, display the public key:

   ```shell
   cat ~/.ssh/id_rsa.pub
   ```

2. Copy the output shown to the clipboard.
3. Switch back to the Gitlab web interface, to the **control-repo** project, to add a Deploy Key:
    1. On the left navigation bar, click **Settings** → **Repository**.
    2. Locate the **Deploy Keys** section and click the **Expand** button next to it.
    3. Set the **Title** to: Puppet Code Manager
    4. Paste the public key in your clipboard to the **Key** field
    5. Click the **Add key** button
4. To validate you can now connect with using the puppet servers keypair, run the following command on the Puppet Enterprise terminal console (substituting the placeholder below with your own gitlab servers FQDN):

   ```markdown
   ssh git@<fqdn of your gitlab server>
   ```

   <div class="noninteractive">

   You should see output similar to the following:

   ```shell
   PTY allocation request failed on channel 0
   Welcome to GitLab, @root!
   Connection to gitlab.company.local closed.
   ```
   </div>

### Step 5: Enable Code Manager on Puppet Enterprise


Code Manager is the easiest way to configure Puppet Enterprise to import automation content from a git repository. In this step you’ll configure code manager to pull code from your Gitlab server.


1. Navigate to the Puppet Enterprise console.
2. On the navigation bar on the left, click on **Node groups**. Then click the **⊞** sign next to **PE Infrastructure** to expand that group.
3. Click on the **PE Master** group and on the page that opens, click the **Classes** tab.
4. Scroll down to the section called **Class: puppet_enterprise::profile::master**
5. Set the first parameter:
    1. Click on the **Parameter name** dropdown box, then find and click on **code_manager_auto_configure**.
    2. Set this parameter’s value to: **true**. Make sure this value is <span style="text-decoration:underline;">not</span> surrounded by quotes.
    3. Click the **Add to node group** button to lock in this parameter.
6. Set the second parameter:
    1. Click on the **Parameter name** dropdown box, then find and click on **r10k_private_key**.
    2. Set this parameter’s value to: 
         ```shell
         /etc/puppetlabs/puppetserver/ssh/id-control_repo.rsa
         ```
    3. Click the **Add to node group** button to lock in this parameter.
7. Set the third parameter:
    1. Click on the **Parameter name** dropdown box, then find and click on **r10k_remote**.
    2. Set this parameter’s value to:
         ```markdown
         git@<fqdn of your Gitlab server>:puppet/control-repo.git
         ```
    3. Click the **Add to node group** button to lock in this parameter.
8. Finally, on the bottom right hand corner of the page, click the button **Commit 3 changes** to save changes.
9. Now you need to run the Puppet Agent on the Puppet Server to configure Code Manager with it’s new parameters. As this particular change will restart a number of Puppet Enterprise services, you should initiate this Puppet run from the terminal console of the Puppet Enterprise server:  

   ```shell
   puppet agent -t
   ```


### Step 6: Deploy the Control Repo to Puppet Enterprise

Code Manager is now able to deploy the Control Repo directly from Gitlab to the Puppet Server. This means that the Control Repos content can be pulled down from source control to the Puppet Server for use in Puppet Enterprise. In this step, you’ll manually deploy the Control Repo using Code Manager for the first time.


1. Deploying code is a privileged action, so you first need to create an RBAC token to allow you to perform the action. Perform this on the terminal console of the Puppet Enterprise server:

   ```shell
   puppet access login --lifetime 1y
   ```


   Enter the credentials for the **admin** account when prompted. A token is now generated & saved to `~/.puppetlabs/token`. This token will be used by default for the `puppet` commands you’ll use next.

2. Perform a Code Manager dry run to ensure everything is working properly:

   ```shell
   puppet code deploy --dry-run
   ```

   <div class="noninteractive">

   You should see output similar to the following: 

   ```shell
   --dry-run implies --all.
   --dry-run implies --wait.
   Dry-run deploying all environments.
   Found 1 environments.
   ```
   </div>

3. Now, perform the initial code deployment for the **production** environment:

   ```
   puppet code deploy production --wait
   ```

   You should see output similar to the following:

   <div class="noninteractive">

    ```shell
    Found 1 environments.
    [
      {
        "deploy-signature": "882089207ccc0f326007c0dbdb415426f6e100f2",
        "environment": "production",
        "file-sync": {
          "code-commit": "81fc54f9a63336b241547b6e96e1d7a33d9a2bfb",
          "environment-commit": "d05e76a7f41608d5eb0756f9b533c9ad156a2f8d"
        },
        "id": 1,
        "status": "complete"
      }
    ]
    ```
    </div>

   Your code is now live on the Puppet Enterprise server, and can be used to manage systems. However, future updates you make to the control repo will have no effect until those updates are deployed to the Puppet Enterprise server again. You _can_ do this manually each time, by re-running the `puppet code deploy production --wait` command, however this quickly becomes tedious. In the next step, you’ll configure a webhook to automate code deployment upon an update (commit) to the Control Repo. Alternatively, this could be automated via a CI/CD system, like Continuous Delivery for Puppet Enterprise (CD4PE).

### Step 7: Enable a webhook for auto sync of your Control Repo updates

As you probably wouldn’t want to manually run the `puppet code deploy` command every time you’ve made a change to your Control Repo on the git server, you can setup a webhook to automate this action.


1. First, you need to enable local webhooks within Gitlab.  You can do this by navigating to **Admin** -> **Settings** -> **Network** -> **Outbound** **Requests** -> **Allow requests to the local network from hooks and services**
2. Next, you need to get the value of the RBAC token, which will form part of the webhook. To get the value, run this command on the terminal console of the Puppet Enterprise server:

   ```shell
   cat ~/.puppetlabs/token
   ```

3. Copy the RBAC token string that is output to the clipboard (be careful to only copy the token).
4. Switch back to the Gitlab web interface, to the **control-repo** project, to add a Webhook:
    1. On the left navigation bar, click **Settings** → **Webhooks**.
    2. Set the **URL** - add the FQDN of your Puppet Server and your RBAC token to the URL below:

         ```markdown
         https://<fqdn of your puppet server>:8170/code-manager/v1/webhook?type=gitlab&token=<paste RBAC token here>
         ```

         Your URL should look something like this:

         ```
         https://puppetserver:8170/code-manager/v1/webhook?type=gitlab&token=0dY2SZHcaJEmFXJOYwBcAK4P0uklvdB0DWUjQvxI64M4
         ```

    3. Uncheck the checkbox in the **Enable SSL verification** option, as you’re using a self-signed certificate
    4. Click the green **Add webhook** button
5. A webhook will appear at the bottom of the page. Click the **Test** button, then select **Push events**. You should get an HTTP 200 result to indicate success.

### Step 8: Add some useful modules to your Puppet Enterprise installation

You now have your Control Repo set up and Code Manager configured, however, the Puppetfile is currently empty. Your Puppetfile is how you’ll express which automation modules you want to download for use within Puppet Enterprise. This step will give you an example Puppetfile that provides a list of modules for the most typical automation use cases on Linux and Windows.


1. Navigate the Gitlab web interface and click on the **control-repo** project.
2. Locate the **Puppetfile** in the root of the repository and click on it. On the page that comes up, click the blue **Edit** button.
3. Replace the content of the file with the following content:


```puppet
# Definition of where the modules come from, only used for backwards compatibility. Code Manager has its own settings for the Forge URL.
forge 'https://forge.puppet.com'

# Useful shared capabilities and common dependencies
mod 'puppetlabs-stdlib',                      :latest
mod 'puppetlabs-apt',                         :latest
mod 'puppetlabs-concat',                      :latest
mod 'puppetlabs-docker',                      :latest
mod 'puppetlabs-exec',                        :latest
mod 'puppetlabs-facts',                       :latest
mod 'puppetlabs-hocon',                       :latest
mod 'puppetlabs-inifile',                     :latest
mod 'puppetlabs-mount_iso',                   :latest
mod 'puppetlabs-powershell',                  :latest
mod 'puppetlabs-pwshlib',                     :latest
mod 'puppetlabs-reboot',                      :latest
mod 'puppetlabs-resource_api',                :latest
mod 'puppetlabs-transition',                  :latest
mod 'puppetlabs-translate',                   :latest
mod 'puppet-archive',                         :latest
mod 'timidri-meltdown',                       :latest
mod 'trlinkin-noop',                          :latest
mod 'pcfens-static_custom_facts',             :latest

# Modules for automation of Puppet Enterprise itself
mod 'puppetlabs-bolt_shim',                   :latest
mod 'puppetlabs-cd4pe',                       :latest
mod 'puppetlabs-cd4pe_jobs',                  :latest
mod 'puppetlabs-device_manager',              :latest
mod 'puppetlabs-puppet_agent',                :latest
mod 'puppetlabs-puppet_authorization',        :latest
mod 'puppetlabs-puppet_conf',                 :latest
mod 'puppetlabs-puppetserver_gem',            :latest

# Modules for displaying Puppet metrics (optional)
mod 'puppetlabs-puppet_metrics_dashboard',    :latest
mod 'puppetlabs-puppet_metrics_collector',    :latest
mod 'puppet-grafana',                         :latest
mod 'puppet-telegraf',                        :latest

# Modules for automating Linux operating systems
mod 'puppetlabs-cron_core',                   :latest
mod 'puppetlabs-host_core',                   :latest
mod 'puppetlabs-k5login_core',                :latest
mod 'puppetlabs-mailalias_core',              :latest
mod 'puppetlabs-mount_core',                  :latest
mod 'puppetlabs-selinux_core',                :latest
mod 'puppetlabs-sshkeys_core',                :latest
mod 'puppetlabs-yumrepo_core',                :latest
mod 'puppetlabs-zfs_core',                    :latest
mod 'puppetlabs-firewall',                    :latest
mod 'puppetlabs-ntp',                         :latest
mod 'puppet-alternatives',                    :latest
mod 'puppet-epel',                            :latest
mod 'puppet-logrotate',                       :latest
mod 'herculesteam-augeasproviders_core',      :latest
mod 'herculesteam-augeasproviders_grub',      :latest
mod 'herculesteam-augeasproviders_ssh',       :latest
mod 'herculesteam-augeasproviders_sysctl',    :latest
mod 'herculesteam-augeasproviders_syslog',    :latest
mod 'herculesteam-augeasproviders_shellvar',  :latest
mod 'herculesteam-augeasproviders_pam',       :latest
mod 'acjohnson-adcli',                        :latest
mod 'saz-timezone',                           :latest
mod 'sgnl05-sssd',                            :latest
mod 'stm-debconf',                            :latest

# Modules for automating Linux applications
mod 'puppetlabs-apache',                      :latest
mod 'puppetlabs-java',                        :latest
mod 'puppetlabs-mysql',                       :latest
mod 'puppet-nginx',                           :latest
mod 'puppet-redis',                           :latest
mod 'biemond-oradb',                          :latest
mod 'biemond-wildfly',                        :latest
mod 'jethrocarr-initfact',                    :latest
mod 'saz-memcached',                          :latest

# Modules for automating Windows operating systems
mod 'puppetlabs-acl',                         :latest
mod 'puppetlabs-chocolatey',                  :latest
mod 'puppetlabs-dism',                        :latest
mod 'puppetlabs-dsc_lite',                    :latest
mod 'puppet-windows_firewall',                :latest
mod 'puppetlabs-registry',                    :latest
mod 'puppetlabs-scheduled_task',              :latest
mod 'puppetlabs-windows_puppet_certificates', :latest
mod 'puppetlabs-wsus_client',                 :latest
mod 'puppet-download_file',                   :latest
mod 'puppet-windows_env',                     :latest
mod 'puppet-windowsfeature',                  :latest
mod 'ayohrling-local_security_policy',        :latest
mod 'encore-powershellmodule',                :latest
mod 'fervid-auditpol',                        :latest
mod 'karmafeast-windows_smb',                 :latest
mod 'nekototori-winrmssl',                    :latest
mod 'noma4i-windows_updates',                 :latest
mod 'jpi-timezone_win',                       :latest
mod 'tse-winntp',                             :latest

# Modules for automating Windows applications
mod 'puppetlabs-iis',                         :latest
mod 'puppetlabs-sqlserver',                   :latest
mod 'kreeuwijk-vmtools_win',                  :latest

```


4. Click the **Commit changes** button.
5. The commit will trigger the webhook, which signals to Code Manager to perform another synchronization of the automation content in your Control Repo. As you’re installing quite a few modules for the first time here, it can take a few minutes for this action to complete. If you want, you can view the progress of the code deployment on the Puppet Enterprise server with this command on the terminal console:
   ``` shell
   tail -f /var/log/puppetlabsæ/puppetserver/puppetserver.log
   ```
6. When the log file above says the following, the deployment has finished:
   <div class="noninteractive">

   ```
   [deploy-pool-1] [p.c.core] Finished deploy attempt for environment 'production'.
   ```
   </div>