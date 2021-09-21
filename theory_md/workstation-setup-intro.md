<div class="tocoutline">

### Table of Contents

<div class="toc">

- [Overview](#overview)
- [Benefits to this approach](#benefits-to-this-approach)
- [Key components](#key-components)
  - [Git](#git)
  - [SSH](#ssh)
  - [VS Code and Puppet extension](#vs-code-and-puppet-extension)
- [Learning Git](#learning-git)
  - [Git overview](#git-overview)
  - [Git in detail](#git-in-detail)
- [Configuring your workstation](#configuring-your-workstation)

</div>

</div>

# Overview<a href="#overview" aria-hidden="true"></a>
The recommended workflow for creating and managing Puppet configurations typically involves users creating configurations on a local copy of the Control Repo, those configurations are then pushed directly to the Control Repo in source control. The Puppet Server maintains a connection with the Control Repo in source control and pulls new versions of the Control Repo either automatically or manually depending on your configuration, via a built-in tool named Code manager. If you've been following the labs in this guide, your Puppet Server will automatically pull a new copy of the Control Repo from source control each time you make and push a change to source control.

You can find an illustration of this workflow below:

<div class="margin">

![alt text for screen readers](../assets/img/control-repo-interaction.png "Workstation workflow")

</div>

# Benefits to this approach<a href="#benefits-to-this-approach" aria-hidden="true"></a>

* It's easier to manage configurations locally on your own workstation rather than your source control providors interface:
    * No need to login to source control everytime you want to make a change.
    * Creating and understanding folder structures within the Control Repo is easier.
    * Leverage VSCode with the Puppet extension to help you create configurations through syntax highlighting and suggestions.
* Transferable skills - This is a common industry standard practice followed by a large majority of developers, DevOps engineers, sys admins and SRE's. 

# Key Components<a href="#key-components" aria-hidden="true"></a>
There are several key components that you'll need to configure to set up your workstation - it's crucial that you understand which function they each play in the workflow that you'll use day to day. This will help you troubleshoot issues faster and allow you to spend more time focusing on creating configurations.

## Git<a href="#git" aria-hidden="true"></a>

Git is a free and open source distributed version control system. It is the basis for the majority of modern source control systems such as Github, Gitlab, Bitbucket, Azure DevOps etc. Git software makes it easy to create configurations on your local workstation and easily push them via directly to the Control Repo in source control. This means that you don’t need to create Puppet configurations elsewhere and manually upload files via your source control’s interface. You _usually_ interact with git software on the command line however, if you're using VS Code you can perform most git actions within the VS Code GUI. 

## SSH<a href="#ssh" aria-hidden="true"></a>

On your workstation, Git will authenticate with your source control provider/instance directly over HTTPS or SSH transport methods however, SSH is recommended due to it’s security benefits over HTTPS. 

## VS Code and Puppet extension<a href="#vs-code-and-puppet-extension" aria-hidden="true"></a>

Setting up VS Code and the Puppet extension on your workstation makes creating Puppet configurations much easier through syntax highlighting, linting, debugging and intellisense capabilities. Once SSH and Git are configured on your workstation, it’s really simple to push your Puppet code configurations from VS Code’s GUI rather than using git commands via the command line. 

# Learning Git<a href="#learning-git" aria-hidden="true"></a>

Before embarking any further, we’d _really_ recommend that you run through the basics of Git, particularly if you aren’t already familiar with Git or source control. If you’re already a Git ninja, you can of course disregard and skip this section.

## Git overview<a href="#git-overview" aria-hidden="true"></a>

The tutorial below will give you a good base level understanding of the fundamentals of what Git is, why you would use it, how it works and how to use it on the command line. Learning Git will not only help when working with Puppet but also any real DevOps/cloud/automation tools in the future. 

#### Git Tutorial Part 1: What is Version Control?
<iframe width="448" height="252" src="https://www.youtube.com/embed/9GKpbI1siow?version=3&vq=hd1080" frameborder="0" allowfullscreen></iframe>

#### Git Tutorial Part 2: Vocab (Repo, Staging, Commit, Push, Pull)
<iframe width="448" height="252" src="https://www.youtube.com/embed/n-p1RUmdl9M?version=3&vq=hd1080" frameborder="0" allowfullscreen></iframe>

#### Git Tutorial Part 3: Installation, Command-line & Clone
<iframe width="448" height="252" src="https://www.youtube.com/embed/UFEby2zo-9E?version=3&vq=hd1080" frameborder="0" allowfullscreen></iframe>

#### Git Tutorial Part 4: GitHub (Pushing to a Server)
<iframe width="448" height="252" src="https://www.youtube.com/embed/ol_UCWox9kc?version=3&vq=hd1080" frameborder="0" allowfullscreen></iframe>

## Git in detail<a href="#git-in-detail" aria-hidden="true"></a>

Once you’ve understood the basics, the step by step tutorial below will cover some of the topics you’ve been introduced to in the Git tutorial above but also walk through some additional Git functionality that you’ll need to in your day to day use with Git.

<a href="https://githowto.com/setup" target="_blank">Githowto</a>

> To set expectations, the Githowto tutorial has 51 steps _but_ they are quite short. We’d recommend setting aside around 1 hour to complete this tutorial comfortably.

# Configuring your workstation<a href="#configuring-your-workstation" aria-hidden="true"></a>