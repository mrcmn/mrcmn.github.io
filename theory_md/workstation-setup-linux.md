<div class="tocoutline">

### Table of Contents

<div class="toc">

  - [Installing Git](#installing-git)
  - [Configuring Git](#configuring-git)
  - [Generating SSH Keys](#generating-ssh-keys)
  - [Copy SSH Public Key Content](#copy-ssh-public-key-content) 
  - [Adding SSH key to source control](#adding-ssh-key-to-source-control)
  - [Cloning the control repository](#cloning-the-control-repository)
  - [Setting up VS Code and Puppet Extension](#setting-up-vs-code-and-puppet-extension)
  - [Push a change through VS Code](#push-a-change-through-vs-code)
  - [Troubleshooting](#troubleshooting)

</div>

</div>

## Installing Git<a href="#installing-git" aria-hidden="true"></a>

In order to interact with a git based repository, you’ll need the git package installed locally on your workstation. It’s available for all platforms, Linux distros, Mac OSX and Windows. 

1. Download and Install Git - You can download Git directly <a href="https://git-scm.com/downloads" target="_blank">here</a>.


## Configuring Git<a href="#configuring-git" aria-hidden="true"></a>

When you commit a change to source control, the author's information must accompany that commit, therefore you cannot push any code until you configure your username and password.

1. To do this, simply run the commands below, substituting the name and email address for your own name and email address:

    ```bash
    git config --global user.name "Joe Bloggs"
    git config --global user.email "joe.bloggs@puppet.com"
    ```

2. You will also need to run these two commands to ensure line endings don’t become an issue for you when working with Git: 

    ```bash
    git config --global core.autocrlf input
    git config --global core.safecrlf warn
    ```

## Generating SSH Keys<a href="#generating-ssh-keys" aria-hidden="true"></a>

We’ll leverage the `ssh-keygen` tool to create a SSH keypair. 

> Ensure you've install Git first as this software includes the relevant binaries required to generate SSH keys on command line.

1. Open a shell window on your workstation and run this command:

    ```bash
    ssh-keygen -t rsa
    ```

2. When asked to "Enter a file in which to save the key", press return/enter.

3. When asked for a passphrase, leave empty and press return/enter and press return once again when asked for confirmation of the passphrase. 

    The command output should look something like this:

    <div class="noninteractive">

    ```
    Your identification has been saved in /home/joebloggs/.ssh/id_rsa.
    Your public key has been saved in /home/joebloggs/.ssh/id_rsa.pub.
    The key fingerprint is:
    SHA256:GI9STe9YbErkwJ51d21YeV3b4L3nhr6Ea880HBONdmI joebloggs@nix3.home
    The key's randomart image is:
    +---[RSA 2048]----+
    |     .. o     .+*|
    |      .*.o. ..o*B|
    |     .oo+.=. .Eo*|
    |     .o* *   o +.|
    |    . o S .   o..|
    |     .       o =.|
    |            . * o|
    |            .* o |
    |           ...=. |
    +----[SHA256]-----+
    ```

    </div>

## Copy SSH Public Key Content<a href="#copy-ssh-public-key-content" aria-hidden="true"></a>

You’ve now generated a SSH keypair. You'll need to provide the public key content to your  source control providor in order to authenticate our workstation. It will do this by verifying the private key (held locally) is the "other half" of the pair that the public key belongs to. Once authenticated, you can clone a copy of the control repo locally and push new changes to source control without the need for additional authentication.

1. Use `cat` against the public key location shown in the command output and copy the public key content:

    ```bash
    cat /home/joebloggs/.ssh/id_rsa.pub
    ```

2. Copy the public key content. It should look something like this:

    <div class="noninteractive">

    ```shell
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCvnyNvotBDzrHud6bSXzWhFH8eu15re
    V67//76iI2JPrvR1lny8bnL9+a1b0NCHjSBHHYggdlOj7jF3IUoD9n7e4jD3ib24qUdYD
    f0Ws6j1hAFk3p2Q1fx1Tf4UPoZ9Mo1m9kGinORdzuGdGxOHqFeV7R1RZM5sF9ty6fSKnw
    VeX8ESRG22X7tl4yI+F5XDA9E6HmKAkDVoVsE/iCfy7fiWQ69KS8gxjXDlzqBLkV9y4zW
    I/43C5T6fCMSwID+81vfsNV42A079o9OW+wq9hO+e2ZC/Ji/etqInBroractgoiih9KyV
    AmJjMGxTsAB0L93Bn66Rju56S3yxdKocjF joebloggs@nix3.home
    ```

    </div>

## Adding SSH key to source control<a href="#adding-ssh-key-to-source-control" aria-hidden="true"></a>

Now that you’ve generated your SSH keys and copied the content of your public key, you need to add it to your source control provider's SSH keys store. 


1. Below you can find documentation on how to add your SSH public key to your source control provider of choice (follow the Gitlab instructions if you’ve been following the labs):

    <a href="https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account" target="_blank">Github</a>

    <a href="https://www.tutorialspoint.com/gitlab/gitlab_ssh_key_setup.htm" target="_blank">Gitlab</a>

    <a href="https://confluence.atlassian.com/bitbucketserver0711/ssh-access-keys-for-system-use-1047557590.html" target="_blank">Bitbucket</a>

    <a href="https://docs.microsoft.com/en-us/azure/devops/repos/git/use-ssh-keys-to-authenticate?view=azure-devops" target="_blank">Azure DevOps</a>

## Cloning the control repository<a href="#cloning-the-control-repository" aria-hidden="true"></a>

Once you’ve added your SSH public key to your source control platform, you should be ready to clone a copy of the control repo via SSH. 

1. Navigate to your control-repo within your source control platform and look for the “clone with SSH” option (or similar phrasing) and copy the link provided.
2. Clone the repo locally to your workstation:

    From your shell, navigate to the directory you wish to place your control repo directory and then type `git clone`, followed by <span style="text-decoration:underline;">your own git repo address</span> that you copied earlier:

    ```markdown
    git clone git@<yoursourcecontrolhostname>:puppet/control-repo.git
    ```

    If the repo was cloned successfully, you should see something like this:

    <div class="noninteractive">

    ```
    git clone git@gitlab.company.com:puppet/control-repo.git
    Cloning into 'control-repo'...
    remote: Enumerating objects: 63, done.
    remote: Counting objects: 100% (63/63), done.
    remote: Compressing objects: 100% (39/39), done.
    remote: Total 3632 (delta 24), reused 50 (delta 18), pack-reused 3569
    Receiving objects: 100% (3632/3632), 442.05 KiB | 9.21 MiB/s, done.
    Resolving deltas: 100% (1782/1782), done.
    ```
    
    </div>

    You should then be able to navigate to the relevant directory to find your control repo locally on your workstation.

## Setting up VS Code and Puppet Extension<a href="#setting-up-vs-code-and-puppet-extension" aria-hidden="true"></a>

Now that you’ve got your control repo downloaded, it’s time to get VS Code and the puppet extension so that we can better interact with the control repo and puppet to create more configurations with ease.

1. Download and install VS Code. You can download VS Code for your workstation <a href="https://code.visualstudio.com/download" target="_blank">here</a>. 
2. Once you’ve downloaded and installed VS Code, you’ll need to download and install the Puppet PDK, which can be found <a href="https://puppet.com/try-puppet/puppet-development-kit" target="_blank">here</a>. It’s a prerequisite for some of the Puppet extensions functionality however, we’ll explore the PDK further in labs coming in the future.
3. Finally, you can install the puppet extension from this <a href="https://puppet-vscode.github.io" target="_blank">page</a>.

We’d recommend that you also check out the <a href="https://puppet-vscode.github.io/docs/features/" target="_blank">Puppet Extension docs</a> once you’re ready to start writing some puppet code.

## Push a change through VS Code<a href="#push-a-change-through-vs-code" aria-hidden="true"></a>

Now that all the prerequisites are out of the way, the control repo is downloaded, git is configured, VS Code and the puppet extension is installed, you are fully ready to make some changes via VS Code to your control repo and push them to source control directly from VS Code.


1. Open VS Code and choose **File** > **Open Folder** then navigate to the control repo click **open**. 
2. Now we’ve opened the control repo, let's edit the **README.md** file and add a line in there with the word “test”.

    <div class="size90margin">

    ![alt text for screen readers](../assets/img/test_string.png "Test string")

    </div>

    Once you’re done, you’ll notice a little dot on the file tab within VS Code. This means that there’s been a new change on this file but it’s not yet saved.

    <div class="size30margin">

    ![alt text for screen readers](../assets/img/file_unsaved.png "Unsaved file")

    </div>

3. To save, simply navigate to **File** > **Save**. You can also press **ctrl + s** on Windows/Linux or **cmd + s** on MacOS. 
4. If everything has been set up correctly, you should now see a “**1**” badge on the source control icon from the sidebar on the left. 

    <div class="size10margin">

    ![alt text for screen readers](../assets/img/source_control_icon.png "Source control icon")

    </div>

    This indicates that there’s now 1 pending change locally in the control repo. Whilst the file has been saved locally, we haven’t pushed any changes directly to source control yet.

5. Click on the badge to see the file with pending change and then click on the **README.md** file under **Changes** to review your change.

    You’ll now see a side by side comparison of the change (before and after).

    <div class="size90margin">

    ![alt text for screen readers](../assets/img/change_details.png "Change details/Git diff")

    </div>

6. If you’re happy with the change, from the sidebar, click the ellipsis(**...**) > **Changes** > **Stage all changes**. You should now see that your changes have moved under the **Stage Changes** category. This means your changes are now ready to commit.

    <div class="size80margin">

    ![alt text for screen readers](../assets/img/stage_all_changes.png "Stage all changes")

    </div>


    <div class="size50margin">
    
    ![alt text for screen readers](../assets/img/changes_staged.png "Commit")

    </div>

7. You can easily add a message along with your commit by typing it in the **Message** field briefly identifying the content of the commit. When you’ve entered your commit message, click the **tick symbol** (✓). Your change is now committed with a message and ready to push.

    <div class="size50margin">

    ![alt text for screen readers](../assets/img/commit.png "Commit")

    </div>

    You’ll now see in the bottom left hand corner, there is 1 commit ready to push upstream to the control repo in source control. 

    <div class="size50margin">

    ![alt text for screen readers](../assets/img/commit_ready.png "Commit ready")

    </div>

8. To push the commit to the control repo, Click the ellipsis(**...**) > **Push.**

    <div class="size50margin">

    ![alt text for screen readers](../assets/img/push.png "Push changes")

    </div>

    You should now see no pending changes to pull or push.

    <div class="size50margin">

    ![alt text for screen readers](../assets/img/no_changes.png "No pending changes")

    </div>

9. Navigate to your source control platform and verify your commit has appeared as expected. It should display your commit message and Git username as configured in the earlier steps:

    <div class="size90margin">

    ![alt text for screen readers](../assets/img/source_control_commit1.png "Commit info in source control 1")

    </div>

    <div class="size90margin">

    ![alt text for screen readers](../assets/img/source_control_commit2.png "Commit info in source control 2")
    
    </div>

## Troubleshooting<a href="#troubleshooting" aria-hidden="true"></a>
* If the `git clone` or `ssh-keygen` command doesn't work, first make sure Git is installed and close your shell window and reopen. If your shell window was open pre-Git installation, it will not have loaded the necessary Git/OpenSSH binaries but should re-load when opening a new shell window. 

