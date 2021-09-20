<div class="tocoutline">

### Table of Contents

<div class="toc">

- [What is the Puppet Forge?](#what-is-the-puppet-forge)
- [What's a module and what does it contain?](#whats-a-module-and-what-does-it-contain)
- [Navigating the Forge](#navigating-the-forge)
  - [Understanding module badges and ratings](#understanding-module-badges-and-ratings)
    - [Supported](#supported)
    - [Partner](#partner)
    - [Approved](#approved)
    - [Tasks](#tasks)
    - [PDK](#pdk)
    - [Quality Score](#quality-score)
- [Forge module Walkthrough](#forge-module-walkthrough)

</div>

</div>

# What is the Puppet Forge?<a href="#what-is-the-puppet-forge" aria-hidden="true"></a>

The Puppet Forge is a catalogue of modules created by Puppet, Puppet partners and community that helps you supercharge and simplify your automation processes. If you’re beginning an automation task without any scripts or other automation content to achieve this currently, the Forge should be your first port of call. 

# What's a module and what does it contain?<a href="#whats-a-module-and-what-does-it-contain" aria-hidden="true"></a>

A module is a term we use to describe a container for pre-built automation. A module will generally contain all the logic and capabilities required for a user to configure a particular software or service, for example <a href="https://forge.puppet.com/modules/puppetlabs/iis" target="_blank">IIS</a> or <a href="https://forge.puppet.com/modules/puppetlabs/apache" target="_blank">Apache</a>. A module can have automation for both desired state configurations as well as ad-hoc automation such as <a href="https://puppet-enterprise-guide.com/theory/tasks-overview.html" target="_blank">Tasks</a> and <a href="https://puppet-enterprise-guide.com/theory/plans-overview.html" target="_blank"> Plans</a>. Modules with desired state capabilities are written in Puppet code and/or Ruby, however Tasks can be written in any scripting language like Bash, Powershell, Python etc. Plans can be written in Puppet code or in YAML.

# Navigating the forge<a href="#navigating-the-forge" aria-hidden="true"></a>

When searching for modules on the Forge, you'll notice a number of "badges" on modules. These badges serve as a guide to help you understanded not just the quality or support level of a given module but also what type of automation the module contains, at a high level. You can find a break down of these module badges below.

## Understanding module badges and ratings<a href="#understanding-module-badges-and-ratings" aria-hidden="true"></a>

### Supported<a href="#supported" aria-hidden="true"></a>

Modules that are supported by Puppet are rigorously tested, will be maintained for the same lifecycle as Puppet Enterprise, and are usually compatible with multiple platforms.

### Partner<a href="#partner" aria-hidden="true"></a>

Modules for a specific partner technology that are rigorously tested with Puppet Enterprise and supported by the partner organization directly (partner licensing may be required).

### Approved<a href="#approved" aria-hidden="true"></a>

Modules that meet Puppet's standards for being well written, reliable, and actively maintained.

### Tasks<a href="#tasks" aria-hidden="true"></a>

Modules that contain Bolt tasks, which provide ad-hoc automation capabilities.

### PDK<a href="#pdk" aria-hidden="true"></a>

Modules that are compatible with Puppet Development Kit (PDK) validation and testing tools.

### Quality Score<a href="#quality-score" aria-hidden="true"></a>

Modules on the Puppet Forge are automatically assigned a score based on Puppet's code standards to help you estimate their quality.


# Forge module Walkthrough<a href="#forge-module-walkthrough" aria-hidden="true"></a>

In the <a href="https://puppet-enterprise-guide.com/theory/forge-example.html" target="_blank">next section</a>, we’ll walk you through how you can effectively leverage module documentation and coded examples to help you to create your own configurations and ensure that you’re getting the most from Forge modules.

