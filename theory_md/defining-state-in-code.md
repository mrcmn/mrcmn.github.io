<div class="tocoutline">

### Table of Contents

<div class="toc">

- [What is Puppet code?](#what-is-puppet-code)
- [Puppet code vs Scripting](#puppet-code-vs-scripting)
    - [Scripting challenges](#scripting-challenges)
    - [Puppet code benefits](#puppet-code-benefits)
- [How does it work?](#how-does-it-work)
  - [Resources](#resources)
    - [What resource types are available?](#what-resource-types-are-available)
    - [Turning resources into configurations](#turning-resources-into-configurations)
  - [Manifests](#manifests)
    - [Location](#location)
    - [Internal layout of manifests](#internal-layout-of-manifests)
  - [Classes](#classes)
    - [Class Definition and Namespacing](#class-definition-and-namespacing)
    - [Adding attributes](#adding-attributes)
    - [Class Declaration](#class-declaration)
    - [Specifying custom attributes via Hiera](#specifying-custom-attributes-via-hiera)
  - [Relationships and Ordering](#relationships-and-ordering)
  - [Pushing configurations to the Puppet Server](#pushing-configurations-to-the-puppet-server)
  - [Apply configuration to node group](#apply-configuration-to-node-group)
  - [Run Puppet to apply configuration](#run-puppet-to-apply-configuration)
  - [Reviewing Reports](#reviewing-reports)
    - [Latest Report](#latest-report)
    - [Historical Node Report](#historical-node-report)
    - [Filter Reports by Run Status](#filter-reports-by-run-status)
    - [Filter Reports by Fact Value](#filter-reports-by-fact-value)
- [Puppet Code Resources](#puppet-code-resources)
- [Examples](#examples)
- [Troubleshooting](#troubleshooting)

</div>

</div>

# What is Puppet code?<a href="#what-is-puppet-code" aria-hidden="true"></a>

Puppet code is an easy way to programmatically describe what configurations should be in place on the operating systems & applications that you want to manage. Puppet code uses a simple desired state language which specifically tries to avoid describing **how** to achieve this goal, instead we only describe **what** the correct end result looks like, allowing Puppet to handle all the steps required to get there. As a consequence, a manifest of Puppet code is not a set of actions to execute, but a definition of the “correct end result”. This is called **desired state**. When you do this at scale across your infrastructure, it's called **Infrastructure-as-Code**. 

# Puppet code vs Scripting<a href="#puppet-code-vs-scripting" aria-hidden="true"></a>

### Scripting challenges

There are some important differences between shell commands and Puppet code.

* Without adding some significant scripting logic to commands, It’s impossible to determine if the commands resulted in changes, or if the system was already correctly configured to begin with.
* If the first command failed but the second command succeeded, the script could return success instead of an error.
* While some commands may be safe to execute on a system that is already correctly configured, this can’t be said for every command out there. Running unnecessary commands could result in unintended consequences like service restarts, software reinstallations or garbled configuration files.
* Scripting logic needs to verify if dependencies are in place before advancing to the next task.
* Commands and configuration details can be different per OS distribution, requiring even more validation to handle each scenario correctly.

### Puppet code benefits

Puppet code was created to solve many of these issues that have traditionally plagued admins. So when you begin to work with Puppet code, the challenges of scripting mentioned above are no longer an issue. 

* Puppet’s desired state enforcement is smart: Puppet actively checks the current state against the desired state before making changes. If the current state already matches the desired state, Puppet won’t make any changes. If there are differences, it will make changes in order to enforce the desired state and report on what changes were made.
* Puppet understands many of the intricacies for achieving the same thing on different flavors of a given OS. For example, if you wanted to install the same package on Red Hat and Ubuntu, the Puppet code to describe that is exactly the same. This allows you as the admin to focus on what really matters (the outcome) vs how to get it done.

# How does it work?<a href="#how-does-it-work" aria-hidden="true"></a>

There are 3 main components needed to create desired state Puppet code configurations:

1. **resources**
2. **manifests**
3. **classes**
 

## Resources<a href="#resources" aria-hidden="true"></a>

<iframe class=video width="448" height="252" src="https://www.youtube.com/embed/fMTQvqZZH7g?version=3&vq=hd1080" frameborder="0" allowfullscreen></iframe>

Resources are the fundamental unit for modeling system configurations. Each resource describes some aspect of a system, like a specific service or package.

The Puppet language codifies the definition of desired state as follows:

<div class="noninteractive">

```
<resource type> { '<name of resource>':
	<attribute> => <desired state of that attribute>
}
```
</div>

For example, if you want the `ntpd` service to be enabled and running at boot time, you would express it like this:


```puppet
service { 'ntpd':
  ensure => 'running',
  enable => 'true',
}
```


Here is the equivalent using an imperative/ad-hoc approach (i.e. command-based):

<div class="noninteractive">

```
systemctl start ntpd
systemctl enable ntpd
```

</div>


> The commands above do not take into account the existing state of the `ntpd` service, so you won’t know whether the service was correctly configured already without building more validation into scripts.

### What resource types are available?<a href="#what-resource-types-are-available" aria-hidden="true"></a>

Every resource type has its own set of attributes that you configure. The Puppet language provides a number of builtin core resources such as `file`, `service`, `package`, `exec` and more. All of the core resources can be found <a href="https://puppet.com/docs/puppet/latest/type.html" target="_blank">here</a>.


Many Puppet modules come with their own resource types that can be leveraged in your Puppet code configurations to more easily manage state in a human readable way. A Puppet module is pre-built Puppet code automation that generally allows you to automate a particular technology, for example: Apache, IIS, SQL server, Splunk and many more. We’ll get more into Puppet modules and the Puppet Forge later in this guide.

For example, the <a href="https://forge.puppet.com/puppetlabs/registry" target="_blank">puppetlabs/registry</a> module gives you `registry_key` and `registry_value` resource types to manage the Windows registry:


```puppet
registry_key { 'HKLM\System\CurrentControlSet\Services\Puppet':
  ensure => present,
}
```

```puppet
registry_value { 'HKLM\System\CurrentControlSet\Services\Puppet\Description':
  ensure => present,
  type   => string,
  data   => "The Puppet Agent service periodically manages your configuration",
}
```

### Turning resources into configurations<a href="#turning-resources-into-configurations" aria-hidden="true"></a>

Now that you've a basic understanding of individual Puppet code resources, it’s time to think about how you make this usable. If you were to navigate to the **Node groups** section of the PE web interface, which is where you assign Puppet code to groups of nodes, you’d see that you can only assign classes, not individual resources. This is because a typical well-managed server has hundreds to even thousands of managed resources; managing them all individually would become extremely complicated.

Classes in Puppet are simply a way of grouping Puppet resources (and logic) together so they become easier to handle. This gives us fewer things to interact with, while any interdependent relationships are managed inside the class itself so you don’t have to worry about it. 

Let's take a look at manifests and classes in more detail.

## Manifests<a href="#manifests" aria-hidden="true"></a>

Manifests are the “home” for Puppet code. Manifests are composed of Puppet code, their filenames use the `.pp` extension. Manifests are also where you define classes. Classes will contain the configurations that you’ll apply to your nodes within Puppet Enterprise. It’s important to understand that there are a few key things to consider when creating Puppet configurations: location, class definition and namespacing. If they aren’t done correctly, Puppet won’t recognise your configurations.

We’ll talk about manifest location first before moving on to class definition and namespacing.

### Location<a href="#location" aria-hidden="true"></a>

Manifests should be stored in a `manifests` subdirectory within a module.

A module is essentially a container of Puppet automation. All Puppet automation content, whether ad-hoc or desired state, is stored within modules.

Modules can be hosted outside of the Control Repo in their own unique repositories (as is the case for any modules you download from the Puppet Forge), or they can be hosted inside the Control Repo, as a simple folder within `site-modules`. 

The `site-modules` directory in the Control Repo is a great starting point for adding your own automation content and the <a href="https://github.com/puppetlabs/control-repo" target="_blank">Puppet Template Control Repo</a> already contains three main directories: `adhoc`, `profile` and `role`. These directories are technically “modules” however the latter two form part of a best practice configuration method named <a href="https://puppet.com/docs/pe/latest/osp/the_roles_and_profiles_method.html" target="_blank">roles and profiles</a>.

<div class="noninteractive">

```markdown
control-repo/
└─ site-modules/
   ├─ adhoc/
   │   ├─ plans/
   │   └─ tasks/
   ├─ profile/
   │   └─ manifests/
   └─ role/
      └─ manifests/
```
</div>

The `profile/manifests` directory is where you put manifests that automate a specific thing by calling some existing automation content, with parameters to control the behavior. The `role/manifests` directory is where you put manifests that call one or more of those profiles, so that you end up with a single configuration that you can assign to one or more servers. 

We’ll get into roles and profiles in more detail in the Going Further section.

When you’re getting started and you’re grouping resources into classes within a manifest, they’ll need to be placed in the `profile/manifests` directory.

**<span style="text-decoration:underline;">Control Repo structure - Example baseline profile</span>**

<div class="noninteractive">

```
control-repo/
└─ site-modules/
   └─ profile/
      └─ manifests/
         └─ baseline.pp
```
</div>

### Internal layout of manifests<a href="#internal-layout-of-manifests" aria-hidden="true"></a>

To be able to write some actual Puppet Code, you need to understand the basic layout of a manifest:

<div class="noninteractive">

```
class <name of class> (
	<data type> $attribute1
	<data type> $attribute2 = <default value>
) {
	<resources and logic>
}
```
</div>

## Classes<a href="#classes" aria-hidden="true"></a>

<iframe class=video width="448" height="252" src="https://www.youtube.com/embed/9vXM5AMBQqc?version=3&vq=hd1080" frameborder="0" allowfullscreen></iframe>

In Puppet, classes are code blocks that can be applied to nodes directly as well as called by code elsewhere. Using classes allows you to reuse Puppet code, and makes reading manifests easier.

### Class Definition and Namespacing<a href="#class-definition-and-namespacing" aria-hidden="true"></a>

A class definition is where the code that composes a class lives. Defining a class makes the class available to be used in other manifests and directly in Puppet enterprise.

In order for Puppet enterprise to recognise a new class, the name of the class must be namespaced accurately in the Puppet manifest. The naming convention goes as follows - **MODULENAME::MANIFESTNAME**

<div class="noninteractive">

```
control-repo/
└─ site-modules/
   └─ profile/
      └─ manifests/
         └─ baseline.pp
```

</div>

Following on from our example above - The **profile** directory in the control repo is technically a module and our manifest is named **baseline**, this means our class definition should be **profile::baseline:**


```puppet
class profile::baseline {
}
```


You can now add resources to this class to form a configuration:


```puppet
class profile::baseline {
  service { 'ntpd':
	ensure => 'running',
    enable => 'true',
  }
}
```


Once this configuration is pushed to the Control Repo and synced with the Puppet server, it will be available as a class in the PE console that can be applied to node groups.

### Adding attributes<a href="#adding-attributes" aria-hidden="true"></a>

Another benefit of using a class to wrap the resources, is that you can give the class its own attributes. These attributes can be various different data types, such as string, integer, array and more - You can find the full list of available attribute types <a href="https://puppet.com/docs/puppet/latest/lang_data_type.html#lang_data_type_list" target="_blank">here</a>.

You can make a simple tweak to the Puppet code manifest to leverage the service name as an attribute. Now if you want to change the name of the service to manage, you can simply do it at the top of the manifest rather than changing the code below. Attributes must be defined as arguments to the class, using parentheses and the target value must refer to the attribute defined above:


```puppet
class profile::baseline (
  String $svcname = 'ntpd'
) {
  service { $svcname:
    ensure => 'running',
    enable => 'true',
  }
}
```


Using attributes allows you to make code inside the class more dynamic, and surface things that you would like to make configurable, instead of hard-coded.  If you want to reuse this class as part of a different configuration you can now do so via class declaration or Hiera.

### Class Declaration<a href="#class-declaration" aria-hidden="true"></a>

A class declaration occurs when a class is called in another manifest. A class declaration tells Puppet to evaluate the code within the called class. Class declarations come in two different flavors: normal and resource-like.

A **resource-like class declaration** occurs when a class is declared like a resource. Using resource-like class declarations allows you to specify class parameters, which override the default values of class attributes like so:


```puppet
class role::linux_build {
  class { 'profile::baseline':		
	svcname => 'sshd',		        
  }
}
```


A **normal class declaration** occurs when the include keyword is used followed by the class name, like so:


```puppet
class role::linux_build {
  include profile::baseline
}
```


When this role class is applied to nodes, Puppet will apply the code within `profile::baseline` with the defaults defined in that profile or any specific configuration from Hiera.

You can also specify multiple classes as part of another manifest such as a role in order to form a “complete configuration”: 


```puppet
class role::linux_build {
  include profile::baseline
  include profile::packages
  include profile::hardening
}
```

<div class="noninteractive">


```markdown
control-repo/
└─ site-modules/
   ├─ profile/
   │   └─ manifests/
   │      ├─ baseline.pp
   │      ├─ packages.pp
   │      └─ hardening.pp
   └─ role/
      └─ manifests/
         └─ linux_build.pp
```

</div>


But we’ll delve into Roles and Profiles later in the Going Further section of the guide.

### Specifying custom attributes via hiera<a href="#specifying-custom-attributes-via-hiera" aria-hidden="true"></a>

In the **Going Further** section, you’ll learn how Puppet’s variable system, Hiera, can automatically populate these class attributes at a common level across all nodes, on a per-node basis, or on any type of grouping criteria you desire. This removes the hardcoding of settings completely, and ensures that anything specific to a system is grabbed from a variable store and applied to the classes that present configurable attributes. In a well-designed system, the Puppet code only contains code and logic, while the Hiera variable system contains the actual configuration values.

Below you can find a _brief_ overview of the what that would look like:

**<span style="text-decoration:underline;">Attribute defined in Puppet class</span>**

<span style="text-decoration:underline;">control-repo/site-modules/profile/manifests/baseline.pp</span>


```puppet
class profile::baseline (
  String $svcname
) {
  service { $svcname:
    ensure => 'running',
    enable => 'true',
  }
}
```


**<span style="text-decoration:underline;">Hiera variable content</span>**

The common Hiera data layer is where generic values live that apply to all nodes; however, as mentioned above, you can get extremely granular with our Hiera right down to the per node level, if needed.

<span style="text-decoration:underline;">control-repo/site-modules/data/common.yaml</span>


```yaml
profile::baseline::svcname: 'sshd'
```


If you applied the `profile::baseline` above to a node group, when Puppet runs, it will automatically look for an attribute match between Puppet code and the Hiera data layer and resolve the `$svcname` attribute value within the Puppet manifest to form a “full configuration”. This allows us to create a separate code and data layer which holds several advantages such as code reusability and the ability to update configuration data without needing to change any code. 

## Relationships and Ordering<a href="#relationships-and-ordering" aria-hidden="true"></a>

<iframe class=video width="448" height="252" src="https://www.youtube.com/embed/gz9Nb6j6dss?version=3&vq=hd1080" frameborder="0" allowfullscreen></iframe>

Relationship metaparameters control the way resources are applied, with the primary use case being to define relationships between dependent resources so that they are applied in the correct order. Resources in manifests are **not** guaranteed to be applied in the order that they’re defined in a manifest. As such, it is usually necessary to explicitly set dependencies between resources via relationship metaparameters. These parameters are as follows:


* **before**: Applies this resource before the target resource.


* **require**: Applies the target resource before this resource.


* **notify**: Applies this resource before the target resource. If this resource changes, a notification is sent to the target resource to refresh itself.


* **subscribe**: Applies the target resource before this resource. If the target resource changes, this resource will refresh itself.


This system can scale up to thousands of resources, with Puppet ensuring things happen in the right order based on the dependencies you set. While scripting has its uses for initial deployments, it’s nearly impossible to achieve the same level of consistency with scripting approaches once systems are running in production. 

You can find a real world example of using relationship metaparameters in the SSH profile example found in the <a href="https://puppet-enterprise-guide.com/theory/configuration-example.html" target="_blank">Configuration Example Walkthrough</a> and in the <a href="https://puppet.com/docs/puppet/latest/lang_relationships.html#lang_rel_metaparameters" target="_blank">Puppet docs</a>.


## Pushing configurations to the Puppet Server<a href="#pushing-configurations-to-the-puppet-server" aria-hidden="true"></a>

The Puppet Server should be configured with a Control Repository stored in source control. This means that configurations should be pushed directly to the Control Repository in source control where it can then be downloaded by the Puppet Server.

Once you’ve created a configuration and pushed it to your Control Repo, it should be available on the Puppet Server within a few seconds. This will happen automatically if the Puppet Server is set to sync with source control each time there’s a commit to the Control Repo (via a webhook). 

Alternatively, you’ll need to run the command `puppet-code deploy --all --wait` directly on the Puppet Server to pull the newest version of the Control Repo containing your configuration.

## Apply configuration to node group<a href="#apply-configuration-to-node-group" aria-hidden="true"></a>

Once the Control Repo syncs with the Puppet Server, your configuration should be available to apply to node groups, within the Puppet Console. 

1. From the sidebar under **Inventory**, click **Node Groups** then click on your target node group.
2. Click on the **Classes** tab and beside **Add new class** enter your class name - for example, `profile::ssh`. Click on your class, and then click the **Add Class** button.

    > If your class doesn’t appear, you may need to click **Refresh**, on the right hand side of the page, to pick up the latest class definitions.

3. In the bottom right corner, click **Commit 1 change**

## Run Puppet to apply configuration<a href="#run-puppet-to-apply-configuration" aria-hidden="true"></a>

Once your configuration has been added to your node group, it’s time to apply it by initiating a Puppet run from the PE console or initiating a Puppet run directly on your target node. There are a few different ways you can do this:

### To perform a Puppet run on a node group:

1. From the PE Console, Navigate to the **Node groups** page and then click on your target node group.
    1. In the top right corner, click **Run > Puppet**.
    2. Under **Job description** add a brief description.
    3. Click **Run job**.

    _Or_

2. From the PE Console, Navigate to the **Jobs** page and click **Run Puppet** in the top right hand corner.
    1. Under **Job description** add a brief description.
    2. Click the dropdown **Select a target type**.
    3. Choose **Node group** and then select or type the name of your target node group then click **Select**.
    4. Click **Run job**.

### To perform a Puppet run on a specific node:

  1. From the PE Console, Navigate to the **Jobs** page and click **Run Puppet** in the top right hand corner.
    1. Under **Job description** add a brief description.
    2. Click the dropdown **Select a target type**.
    3. Choose **Node list** and then type the name of your target node, click **Search** then click on your node.
    4. Click **Run job**.

2. Log on to your target node via SSH/RDP, open a shell window/command prompt and type **puppet agent -t**.


## Reviewing Reports<a href="#reviewing-reports" aria-hidden="true"></a>

You can review your node's Puppet agent report in order to check the specific resources with changes on this Puppet run and will also help you understand whether there’s been any new <span style="text-decoration:underline;">Intentional configuration changes</span> or if Puppet has made <span style="text-decoration:underline;">corrective configuration changes</span>. Corrective changes can signal that your targets have experienced configuration drift and although Puppet has fixed that, it may be something to investigate if the same corrective changes occur over a period of time.

### Latest Report

1. Navigate to the **Status** page to find an overview of the latest Puppet run information across the estate.
2. To view an individual node's Puppet run report, under the heading **Last Report,** click on the timestamped link next to your node, for example: **2021-07-28 14:25 Z**

### Historical Node Report

1. Navigate to the **Nodes** page, then click on your node.
2. Click on the **Reports** tab.
3. View the relevant report by clicking on the timestamped link, for example: **2021-07-28 14:25 Z**

### Filter Reports by Run Status

1. Navigate to the **Reports** page
2. Filter reports by Event status by choosing an option from the drop down **Filter by run status**
3. View the relevant report by clicking on the timestamped link, for example: **2021-07-28 14:25 Z**

### Filter Reports by Fact Value

1. Navigate to the **Reports** page
2. You can also filter reports by fact value by clicking **Filter by fact value** and entering your fact criteria and clicking **Add**. We've included some examples below:

    | Action                                            | Filter                       | 
    | -----------                                       | -----------                  |
    | Reports from nodes running Ubuntu                 | _operatingsystem = Ubuntu_   |
    | Reports from nodes running Windows                | _operatingsystem = windows_  |
    | Reports from nodes with FQDN containing “.domain” | _fqdn != .domain_            |

4. You can then filter by Event status by choosing an option from the drop down **Filter by run status**
3. View the relevant report by clicking on the timestamped link, for example: **2021-07-28 14:25 Z**

# Puppet Code Resources<a href="#puppet-code-resources" aria-hidden="true"></a>
There are number of helpful resources when you start writing Puppet code, the vast majority can be found within the official Open Source Puppet documentation. 

> Whilst most coding practices described in the Open Source Puppet docs will be transferrable to Puppet Enterprise, not everything is appropriate in the context in Puppet Enterprise.

You may not need _all_ of these resources in the very beginning, so don't worry about trying to learn every aspect of each of the items listed, however you should bookmark them for future reference. Here are a selection of key topics you'll need:

 <a href="https://puppet.com/docs/puppet/latest/puppet_language.html" target="_blank">The Puppet language</a>

 <a href="https://puppet.com/docs/puppet/latest/lang_visual_index.html" target="_blank">Puppet language syntax examples</a>

<a href="https://puppet.com/docs/puppet/latest/lang_resources.html" target="_blank">Resources</a>

<a href="https://puppet.com/docs/puppet/latest/lang_defined_types.html" target="_blank">Defined resource types</a>

<a href="https://puppet.com/docs/puppet/latest/lang_classes.html" target="_blank">Classes</a>

<a href="https://puppet.com/docs/puppet/latest/lang_relationships.html" target="_blank">Relationships</a>

<a href="https://puppet.com/docs/puppet/latest/lang_variables.html" target="_blank">Variables</a>

<a href="https://puppet.com/docs/puppet/latest/lang_conditional.html" target="_blank">Conditional Statements and Expressions</a>

<a href="https://puppet.com/docs/puppet/latest/lang_data.html" target="_blank">Data Types</a>

<a href="https://puppet.com/docs/puppet/latest/powershell-dsc-resources.html" target="_blank">Powershell DSC Resources</a>

<a href="https://puppet.com/docs/puppet/latest/lang_functions.html" target="_blank">Functions</a>

<a href="https://puppet.com/docs/puppet/latest/function.html" target="_blank">Function References</a>



# Examples<a href="#examples" aria-hidden="true"></a>
You can find a basic Puppet code example within the <a href="https://puppet-enterprise-guide.com/theory/configuration-example.html" target="_blank">Configuration Example Walkthrough</a> page but you can find more examples on the <a href="https://github.com/kinners00/puppet-examples/tree/production/examples/puppet-code" target="_blank">Puppet examples repo</a>.

> Examples in the repo above are resources within manifests. If you want to use these configurations, you'll need to wrap them in a class within a manifest and add the manifest to a modules `manifests` subdirectory, such as `profile/manifests`. See [Class Definition and Namespacing](#class-definition-and-namespacing) above for more information.


# Troubleshooting<a href="#troubleshooting" aria-hidden="true"></a>

If your class _doesn’t_ appear in the PE console from the Classes tab of your target node group, there could be a few potential problems:

1. **You need to refresh class definitions**

    Within your node group, under the Classes tab, ensure that class definitions are up to date. Class definitions are refreshed automatically every 10 minutes, however this value can be customised via the <a href="https://puppet.com/docs/pe/latest/config_console.html#console_and_console_services_parameters" target="_blank">`puppet_enterprise::profile::console::classifier_synchronization_period` parameter</a>. If you’re trying to apply new configurations to node groups straight after pushing changes to the Control Repo, you may need to click “refresh” to manually update class definitions as the refresh interval hasn’t been triggered yet. If your class doesn’t appear after a refresh, review the points below.
2. **Your manifest isn’t in a location where Puppet looks for them**

    See [Location](#location) above.

3. **Incorrect manifest class definition**

     If your manifest is in the correct location, ensure your class definition at the top of the manifest corresponds to the **MODULENAME::MANIFESTNAME** naming convention - see [Location](#location) and [Class Definition and Namespacing](#class-definition-and-namespacing) above.

4. **Your Puppet Code is not syntactically correct and is therefore not valid**

     Ensure that the syntax in your Puppet manifest is aligned with the examples above and review your manifests against the examples found on <a href="https://puppet.com/docs/puppet/latest/lang_visual_index.html" target="_blank">Puppet language syntax examples</a> page. We would also recommend you use an IDE such as VS Code with the Puppet Extension. This will help keep you on track when writing code to ensure that Puppet code is valid and syntactically correct. If you want to set up and configure VS Code with the Puppet extension, we walk through this process in <a href="https://puppet-enterprise-guide.com/theory/workstation-setup.html" target="_blank">Workstation Setup</a>.

5. **You need to perform a Code Deploy on the Puppet Server**

    Ensure code has been deployed to the Puppet Server. If you don't have webhooks configured to automatically trigger code deployment, you'll need to run the `puppet-code deploy` command on the Puppet Server - you can find more info on how to perform a manual code deploy <a href="https://puppet.com/docs/pe/latest/puppet_code.html#deploy_environments" target="_blank">here</a>.

6. **There’s a connection issue between your Workstation, Control Repo or Puppet Server**

    If you’re pushing configurations locally from your own workstation, ensure that commits are successfully syncing with your source control provider, you can do this by logging on to your chosen platforms web console and check that commits are up to date. If this is working, you’ll then need to check the SSH connection between the Puppet Server and your source control platform. The issue may lie with your SSH keys or firewall configuration on the Primary Puppet Server and/or your source control providor. If you’ve been following this guide, verify you’ve opened the relevant ports as described in <a href="https://puppet-enterprise-guide.com/labs/lab-install-pe.html" target="_blank">LAB: Install PE</a> and that you've followed the steps in <a href="https://puppet-enterprise-guide.com/labs/lab-set-up-the-control-repo.html" target="_blank">LAB: Set up the Control Repo</a>. If you’re still having trouble, check out <a href="https://puppet.com/docs/pe/latest/code_mgr_config.html" target="_blank">Configuring Code Manager</a> in the PE docs. <a href="https://puppet.com/docs/pe/latest/code_mgr_troubleshoot.html" target="_blank">Troubleshooting Code Manager</a> can also be a useful resource when troubleshooting code deployment issues.



