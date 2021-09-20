<div class="tocoutline">

### Table of Contents

<div class="toc">

- [What is a Plan?](#what-is-a-plan)
- [Use Cases](#use-cases)
- [How do they work?](#how-do-they-work)
  - [Components of a Plan](#components-of-a-plan)
  - [What happens during a Plan run?](#what-happens-during-a-plan-run)
  - [Format](#format)
  - [Internal layout of Puppet Plan Manifests](#internal-layout-of-puppet-plan-manifests)
  - [Storage](#storage)
  - [Naming](#naming)
  - [Parameters](#parameters)
  - [Plan Metadata](#plan-metadata)
  - [Plan Functions](#plan-functions)
- [Running Plans in PE](#running-plans-in-pe)
- [Examples](#examples)
- [Troubleshooting](#troubleshooting)

</div>

</div>

# What is a Plan?<a href="#what-is-a-plan" aria-hidden="true"></a>

A Puppet Plan allows you to execute a series of steps in sequential order to achieve an overall workflow. Within a Plan, you can use a combination of commands, scripts, tasks, Puppet Code and more to achieve a full end to end workflow. This means that not only can you leverage your existing investments but you can also leverage the Puppet Forge for thousands of modules containing pre-built, ready to use Puppet automation content.

# Use Cases<a href="#use-cases" aria-hidden="true"></a>

* Complex Workflows such as:
    * Multi-tiered Application Deployment
    * COTS application upgrades
    * Patching including pre/post patching tasks

# How do they work?<a href="#how-do-they-work" aria-hidden="true"></a>

Plans can be as simple as chaining together multiple scripts or Tasks to achieve a goal or they can be as complex and powerful as you’d like them to be. As the execution of a Plan is procedural (i.e. it runs line-by-line from top to bottom), they are fairly easy to read, understand and develop.

## Components of a Plan<a href="#components-of-a-plan" aria-hidden="true"></a>

The Puppet Plan Manifest describes the various steps needed to achieve an overall workflow. Each step in a Plan describes which action needs to take place and on which target(s) that step should be executed. There are various types of steps that can be included as part of a Plan. These are known as <a href="https://puppet.com/docs/bolt/latest/plan_functions.html" target="_blank">Plan functions</a>. Here are a few key ones that you'll most likely use day to day:

  * Commands (run_command)
  * Scripts (run_script)
  * Tasks (run_task)
  * Puppet Code (apply and apply_prep)
  * Upload file (upload_file)
  * Download file (download_file)

## What happens during a Plan run?<a href="#what-happens-during-a-plan-run" aria-hidden="true"></a>

1. Plans execute directly on the Puppet Server. There is only ever 1 instance of a Plan, however steps inside a Plan may have external targets. In this case, those steps will execute remotely & concurrently against their targets. The Puppet Server will copy relevant content to remote targets such as Task content and then execute them directly on the targets. This can be done agentlessly via SSH or WinRM or via the Puppet Agent transport, PCP. 
2. Once the steps have been executed on targets, the output is returned to the Puppet Server where it can then be viewed within Puppet Enterprise console.

Credentials aren't required for any steps in the Plan where the targets are Puppet Agent nodes. This is made possible by the Puppet Agent, which is already authenticated with Puppet Server and has full privileges to make changes on nodes. However, Role based access control (RBAC) ensures that these privileges cannot be abused. RBAC can used to limit Plan execution to ensure teams can only run Plans that are specific to their team. 

## Format<a href="#format" aria-hidden="true"></a>

Plans can be written in either YAML or the Puppet Language. This means that Plans can have either the extension **.yaml** or **.pp.**. In this guide we are primarily focused on the Puppet Language as this method is more flexible and powerful in comparison to YAML. 

When working with the Puppet Language, we’d recommend you use a good text editor like VS Code with the Official Puppet extension to keep you on track with syntax, formatting etc.

## Internal layout of Puppet Plan Manifests<a href="#internal-layout-of-puppet-plan-manifests" aria-hidden="true"></a>

Before you create Plans, you’ll need to understand how Plan Manifests are structured. Plan Manifests follow a similar layout to Puppet Code Manifests as you’ll see in the next section of this guide. Puppet Language based plans are more powerful than YAML plans as they can take advantage of the power of the Puppet Language for more advanced logic and error handling.

<div class="noninteractive">

```puppet
plan <name of plan> (
      <data type> $parameter1
	  <data type> $parameter2 = <default value>
) {
	  <plan step/function>
      <plan step/function>
      <plan step/function>
}
```
</div>


## Storage<a href="#storage" aria-hidden="true"></a>

Plans will be recognized and available in Puppet Enterprise if they are contained within a `Plans` subdirectory of a module, in your Control Repo. A module is a container for Puppet automation in Puppet Enterprise. We’ll go more in depth on modules and their uses in the further sections.

Within the Puppet Template Control Repo, the `adhoc` directory is technically a module so you can create a Plan and add it to your Control Repo under **site-modules/adhoc/plans**. 

<div class="noninteractive">

```
├─ controlrepo/
    └─ site-modules/
        └─ adhoc/					
            └─ plans/
                └─ workflow.pp		      
```
</div>

## Naming<a href="#naming" aria-hidden="true"></a>

Plan Manifest filenames can only use:

* Lowercase characters
* Underscores
* Numbers

#### Namespacing

For Plans, Puppet uses the namespacing syntax: 

**MODULENAME::PLANNAME**

<div class="noninteractive">

```
├─ controlrepo/
    └─ site-modules/
        └─ adhoc/					
            └─ plans/
                └─ workflow.pp		      
```
</div>

The **adhoc** directory is a module so this means that our Plan example shown above will be namespaced as:

**adhoc::workflow**

In order for Puppet Enterprise to recognize a plan, <span style="text-decoration:underline;">the name of the plan must be accurately defined at the top of the manifest.</span>

Within the `workflow.pp` manifest, the Plan name at the top must correspond to this naming convention described above:

```puppet
plan adhoc::workflow {
}
```

If you’re not using the Puppet Template Control Repo or want to create a separate module for your Plans, you can simply add a directory within **site-modules** with a **plan** subdirectory and then add your plan:

**Module Example (ops_team)**

<div class="noninteractive">

```
control-repo/
└─ site-modules/
   └─ ops_team/
      └─ plans/
         └─ workflow.pp
```
</div>

However, you must ensure that you modify the Plan name within your manifest to account for the new module location like so:


```puppet
plan ops_team::workflow {
}
```

## Parameters<a href="#parameters" aria-hidden="true"></a>

Input parameters are specified at the top of plans. Puppet Plans will always execute directly on the Puppet server, but for each individual plan step you can specify other nodes as targets to perform the action on. If your Plan will perform multiple steps against the same set of targets, you can add a `$targets` parameter to the Plan, allowing users to specify the target set just once when running it from the PE console. Plans differ from Tasks in that each step could have different targets and some steps might not have targets at all. 


In the example below, we've created a input parameter to allow end users to specify a single set of targets for this Plan:


```puppet
plan adhoc::workflow(
  TargetSpec $targets,
) {
}
```

> When adding parameters to Plans, you'll need open parenthesis after the Plan name/definition and close them after your last parameter.

## Plan Metadata<a href="#plan-metadata" aria-hidden="true"></a>

Plan Metadata helps users of a Plan easily understand what the Plan does and what parameters it accepts. In Tasks, we do this via Task Metadata, but with Plans, we can add descriptions directly in the Plan itself. 

> Plan Metadata (unlike Task Metadata) does not enforce parameters. Parameters and their accepted inputs are enforced by their definition at the top of the Plan Manifest. 

Descriptions for a Plan and it’s parameters must be specified **above** the Plan name/definition.

To add an overall description for the plan, use the @summary string followed by a description:


```puppet
# @summary this is the description of my plan
plan adhoc::workflow(
  TargetSpec $targets
){
}
```


If you want to add a description for each of the parameters in the Plan, you use the @param string followed by the name of the parameter and then it’s description:


```puppet
# @summary This is the description of my plan
# @param targets The targets to run on.
plan adhoc::workflow(
  TargetSpec $targets
){
}
```


## Plan Functions<a href="#plan-functions" aria-hidden="true"></a>

The power of Plans comes into play when create a complex workflow, made up of multiple types of automation steps, known as Plan Functions. Plan Functions run in a procedural order from top to bottom. If there are any issues with a given step, the Plan will fail at that stage. This type of automation can be useful in scenarios where automation steps need to run in a specific order to achieve a goal, such as application deployments or patching cycles.

There are many <a href="https://puppet.com/docs/bolt/latest/plan_functions.html" target="_blank">Plan Functions</a> that can be used within Plans, including Puppet Code. You can find a trivial example below of Plan syntax when chaining together multiple Plan functions:


```puppet
# @summary This is the description of my plan
# @param targets The targets to run on.
plan adhoc::workflow(
  TargetSpec $targets,
) {
  write_file('Hello, world!', '/Users/Joe/hello.txt', $targets)
  out::message('File created!')

  run_command('cat Users/Joe/hello.txt', $targets)
  run_task('adhoc::delete_file', $targets,)
  out::message('File Deleted!')
}
```

# Running Plans in PE<a href="#running-plans-in-pe" aria-hidden="true"></a>

Once the Control Repo syncs with the Puppet Server, your Plan should be available in the Puppet Console.

1. From the sidebar under **Orchestration**, click **Plans** then **Run a plan**.
2. Under **Plan** in the _Enter plan to run_ textbox, you can click and then scroll to find your Plan.
3. If your Plan has metadata, you can display it by clicking **view plan metadata**.
4. Below, fill in your relevant parameters, if you have any. 
5. Choose targets by selecting them from a **Node List**, **PQL Query** or **Node Group**.
6. Once you’ve chosen your targets, click **Run Plan**.

# Examples<a href="#examples" aria-hidden="true"></a>
Aside from the examples shown above, you can find some further Plan examples within the <a href="https://puppet-enterprise-guide.com/theory/plan-example.html" target="_blank">Plan Example Walkthrough</a> page and the <a href="https://github.com/kinners00/puppet-examples/tree/production/examples/plans" target="_blank">Puppet Examples repo</a>.

# Troubleshooting<a href="#troubleshooting" aria-hidden="true"></a>

Once you've added your Plan to the Control Repo, if it doesn’t appear in the PE console after a minute or so (could be longer for a larger code base), there could be a few potential problems:


1. **Your Plan isn’t in a location that Puppet looks for Plans**

    Refer to [Storage](#storage) section of this page.

2. **Incorrect Plan Manifest definition**

    If your Plan is in the correct location, ensure your Plan definition at the top of your Puppet language plan corresponds to the **MODULENAME::PLAN** naming convention - see [Naming](#naming).

3. **Your Plan is not syntactically correct and is therefore not valid**

    Review examples in [Storage](#storage) and [Naming](#naming) and consider using an IDE such as Visual Studio code with the Puppet Extension. This will help keep you on track when writing Puppet Code, Tasks and Plans to ensure that syntax is valid.

4. **You need to perform a Code Deploy on the Puppet Server**

    If you’re pushing configurations locally from your own workstation, ensure that commits are successfully syncing with your source control provider, you can do this by logging on to your chosen platforms web console and check that commits are up to date. If this is working, you’ll then need to check the SSH connection between the Puppet Server and your source control platform. The issue may lie with your SSH keys or firewall configuration on the Primary Puppet Server and/or your source control providor. If you’ve been following this guide, verify you’ve opened the relevant ports as described in <a href="https://puppet-enterprise-guide.com/labs/lab-install-pe.html" target="_blank">LAB: Install PE</a> and that you've followed the steps in <a href="https://puppet-enterprise-guide.com/labs/lab-set-up-the-control-repo.html" target="_blank">LAB: Set up the Control Repo</a>. If you’re still having trouble, check out <a href="https://puppet.com/docs/pe/latest/code_mgr_config.html" target="_blank">configuring code manager</a> in the PE docs. <a href="https://puppet.com/docs/pe/latest/code_mgr_troubleshoot.html" target="_blank">Troubleshooting Code Manager</a> can also be a useful resource when troubleshooting code deployment issues.