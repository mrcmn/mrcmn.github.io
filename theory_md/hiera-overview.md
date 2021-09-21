<div class="tocoutline">

### Table of Contents

<div class="toc">

- [Description](#description)
- [Why would you use it?](#why-would-you-use-it)
- [How does it work?](#how-does-it-work)
  - [Data hierarchy](#data-hierarchy)
  - [Hiera data layers](#hiera-data-layers)
    - [Common data layer](#common-data-layer)
    - [Nodes data layer](#nodes-data-layer)
    - [Adding custom data layers](#adding-custom-data-layers)
  - [Referring to Hiera in code](#referring-to-hiera-in-code)
  - [Specifying attributes in Hiera](#specifying-attributes-in-hiera)
    - [Namespacing](#namespacing)
    - [Attributes in the common data Layer](#attributes-in-the-common-data-layer)
    - [Attributes in the nodes data Layer](#attributes-in-the-nodes-data-layer) 
  - [Testing Hiera lookups](#testing-hiera-lookups)

</div>

</div>

# Description<a href="#description" aria-hidden="true"></a>

Hiera is the builtin way that Puppet allows you to define variables for individual servers, or groups of servers. It's called Hiera due the hierarchical structure that it lets you define. You define layers of variables, from the most-specific (e.g. per server) to the least-specific (e.g. common/default). 

When using Hiera, Puppet code and it’s associated logic lives in manifests, structured as roles and profiles. 

Infrastructure specific data lives in Hiera and is consumed via lookups in Puppet code.

# Why would you use it?<a href="#why-would-you-use-it" aria-hidden="true"></a>

Using Hiera allows code to remain generic, reusable, shareable and scalable. Instead of hardcoding data values in Puppet code, you can make these data points variables which then look for specific data in Hiera in order to form a “complete” configuration.

# How does it work?<a href="#how-does-it-work" aria-hidden="true"></a>

## Data hierarchy<a href="#data-hierarchy" aria-hidden="true"></a>

The `hiera.yaml` file is where you define which layers you want and what the hierarchy of those layers should be. Think of this as the “control plane” for Hiera.

Below you can find a basic example of a `hiera.yaml` file. The data layers and their hierarchy are defined under the `paths` key:


```yaml
---
version: 5

defaults:
  datadir: "data"

hierarchy:
  - name: 'Yaml backend'
    data_hash: yaml_data
    paths:
      - "nodes/%{trusted.certname}.yaml"
      - 'common.yaml'
```


As you can see from this default hierarchy, data values will be consumed in this order:


1. `data/nodes/%{trusted.certname}.yaml`
2. `data/common.yaml`

This means that by default, node specific values identified by a YAML file with their certname as the file name, will take precedence over the `common.yaml` layer.

For example, if the same Hiera value is found in both the `nodes` data layer and the `common` layer, the value within the `nodes` layer will be used and the value within the `common` layer will be ignored. This is because Hiera defaults to the `unique` type lookup which uses the first successful lookup of a variable through the layers of the hierarchy, top to bottom. Since the `nodes` layer is a level above the `common` layer in the Hiera hierarchy, the variable set in the upper layer will be chosen. There are other <a href="https://puppet.com/docs/puppet/latest/hiera_merging.html" target="_blank">lookup types</a> that can combine and merge the lookup results from multiple layers together, and this can be configured on a very granular level.

## Hiera data layers<a href="#hiera-data-layers" aria-hidden="true"></a>

The Hiera `data` directory and it’s subdirectories house the actual YAML files that contain the variables for your configuration:

<div class="noninteractive">

```
control-repo/
└─ data/            		       <- This is where all your variables go
   ├─ common.yaml                  
   └─ nodes/     
```

</div>

### Common data layer<a href="#common-data-layer" aria-hidden="true"></a>

The common data layer is the lowest layer of the hierarchy. This is where generic and default values that apply to all nodes should live.

<div class="noninteractive">

```
control-repo/
└─ data/            		  
   ├─ common.yaml                   <- Generic default values go here  
   └─ nodes/      		     
```

</div>

### Nodes data layer<a href="#nodes-data-layer" aria-hidden="true"></a>

The nodes layer is where you can specify data values exclusive to specific nodes. Simply create a YAML file within the `nodes` subdirectory with the certname of your target node in Puppet Enterprise as it’s file name and specify the relevant attributes and values within that file:

<div class="noninteractive">

```
control-repo/
└─ data/
   ├─ common.yaml             		  
   └─ nodes/        		  
   	  └─ mynodecertname.yaml        <- Node specific values go here
```

</div>

### Adding custom data layers<a href="#adding-custom-data-layers" aria-hidden="true"></a>

You can easily create your own custom Hiera data layer based on any of the facts that Puppet gathers on a node, for example, if you wanted to apply the same configuration to two nodes, of two different OSes with slightly different values, you could create an **os** data layer that would supply a different value based on the `operatingsystem` fact of the target node.

First, you'll need to create the relevant data directory to house the specific operating system Hiera values:

<div class="noninteractive">

```
control-repo/
└─ data/
   ├─ common.yaml   	            		  
   ├─ nodes/  
   └─ os/                   <- New data directory
      └─ RedHat.yaml        <- OS specific values go here      
```

</div>

Now we need to define the new data layer within the paths key in the `hiera.yaml` hierarchy.

The syntax for specifying a data layer is `"subdirectoryname/%{facts.factname}.yaml"`

So because our data subdirectory is named `os` and the fact name is `operatingsystem` our data layer definition should be `"os/%{facts.operatingsystem}.yaml"`. 

> Puppet facts are case sensitive, so the YAML filename(s) should match the case of the expected fact value(s).

We can simply add this above the `common.yaml` layer:


```yaml
---
version: 5

defaults:
  datadir: "data"

hierarchy:
  - name: 'Yaml backend'
    data_hash: yaml_data
    paths:
      - "nodes/%{trusted.certname}.yaml"
      - "os/%{facts.operatingsystem}.yaml"
      - 'common.yaml'
```

Custom data layers can be useful when leveraging some of the built-in Puppet facts, but to make configurations and Hiera really “work” for your organisation it may be more useful to create some custom facts tailored to business specific needs and then create a Hiera data structure based around those. 

Custom external facts are the easiest to get started with as you can create them using any scripting language. Regular custom facts are written in Ruby. You can find out more about both types in puppet docs: <a href="https://puppet.com/docs/puppet/latest/external_facts.html" target="_blank">External facts</a> and <a href="https://puppet.com/docs/puppet/latest/custom_facts.html" target="_blank">custom facts</a>.

## Referring to Hiera in code<a href="#referring-to-hiera-in-code" aria-hidden="true"></a>

We’ve covered the data layer and hierarchy so far, but how do we actually tell Puppet to fetch values from Hiera rather than hard coding data into Puppet manifests? 

We simply add attributes to the class definition within parentheses at the top of a manifest and reference the value in the resource to this attribute:

### Hardcoded parameters without Hiera

<span style="text-decoration:underline;">control-repo/site-modules/profile/manifests/baseline.pp</span>


```puppet
class profile::baseline {

  service { 'sshd':
    ensure => 'running',
    enable => 'true',
  }
}
```


### Hiera attribute within class

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


## Specifying attributes in Hiera<a href="#specifying-attributes-in-hiera" aria-hidden="true"></a>

So we’ve learnt how to structure the Hiera data hierarchy, how to refer to Hiera data in puppet code but now we need to know how to actually create Hiera data that Puppet configurations can use. To do this, first we need to learn about namespacing. 

### Namespacing<a href="#namespacing" aria-hidden="true"></a>

In order for Hiera to automatically assign the correct value to the correct class, we need to specify the class name with the variable name within the corresponding data layer. The syntax is as follows:

<div class="noninteractive">

```yaml
CLASSNAME::HIERAATTRIBUTE: MYVALUE
```

</div>

### Attributes in the common data Layer<a href="#attributes-in-the-common-data-layer" aria-hidden="true"></a>

We can simply specify this within the <span style="text-decoration:underline;">common.yaml</span> file if we want to add a generic value that applies to all nodes. Notice the namespacing below - our Hiera attribute is specified in the `profile::baseline` class, is named `$svcname` and in this instance we want the value to default to `'sshd'` - this is how it should look:

<span style="text-decoration:underline;">control-repo/site-modules/data/common.yaml</span>


```yaml
profile::baseline::svcname: 'sshd'
```

<div class="noninteractive">

```
control-repo/
└─ data/            		      
   ├─ common.yaml                  
   └─ nodes/     
```

</div>


### Attributes in the nodes data Layer<a href="#attributes-in-the-nodes-data-layer" aria-hidden="true"></a>

We can get extremely granular with our Hiera variables right down to a “per node” level, if needed.

The syntax found at the nodes layer is as follows:

 `"nodes/%{trusted.certname}.yaml"`

This means that we need to create a yaml file with the name of the target node’s certname (name of the nodes certificate name within PE) within the `nodes` subdirectory of the `data` directory. Once you’ve created this file, you can then add your Hiera variable content.

For example, if your target nodes certname within PE is **prodnode1.company** then you’ll need to create a YAML file named <span style="text-decoration:underline;">prodnode1.company.yaml</span> and add your relevant Hiera attribute content. In this example, the Hiera attribute value is `'ntpd'`:

<span style="text-decoration:underline;">control-repo/site-modules/data/nodes/prodnode1.company.yaml</span>


```yaml
profile::baseline::svcname: 'ntpd'
```

<div class="noninteractive">

```
control-repo/
└─ data/
   ├─ common.yaml          		  
   └─ nodes/        		  
   	  └─ prodnode1.company.yaml
```
</div>



Regardless of the content of the `common.yaml` file, the Hiera value found in the `nodes` layer has <span style="text-decoration:underline;">priority</span> due to the data hierarchy configured within the  `hiera.yaml` (shown here) file and values within this layer will be consumed first if there are values found for the same variable in both the `common.yaml` and <span style="text-decoration:underline;">prodnode1.company.yaml</span>.  

Now If we applied the `profile::baseline` above to a node group containing the node <span style="text-decoration:underline;">prodnode1.company.yaml</span>, when Puppet runs, it will automatically look for an attribute match between Puppet code and the Hiera data layer and resolve the `$svcname` attribute value within the Puppet manifest to form a “full configuration”. 

Below you can find the “result” of the `profile::baseline` when the Hiera values have been resolved based on the node the configuration is applied to.

### Resolved value for “common” node

<div class="noninteractive">

```yaml
'sshd'
```
</div>

### Resolved value for prodnode1.company

<div class="noninteractive">

```yaml
'ntpd'
```

</div>


## Testing Hiera lookups<a href="#testing-hiera-lookups" aria-hidden="true"></a>

Once you’ve defined a Hiera variable in your Puppet manifest and specified a value within the Hiera data layer, you can quickly test if Hiera is returning the expected value for a given node, without the need to simulate or execute a puppet run.

To do this, you’ll need to log on to the Primary Puppet server via SSH and run the Puppet lookup command. The command syntax is as follows:


```
puppet lookup <variable> --node <certname>
```


So if you wanted to check if the `svcname` hiera variable resolves to the correct value for the node <span style="text-decoration:underline;">prodnode1.company</span>, you would need to run the following command:


```
puppet lookup profile::baseline::svcname --node prodnode1.company
```


### Output

<div class="noninteractive">

```yaml
--- ntpd
```
</div>

If you want to understand exactly where a value is coming from within the hiera data layer, you can add the flag `--explain` to your command:


```
puppet lookup profile::baseline::svcname --node prodnode1.company --explain
```


### Output

<div class="noninteractive">

```
   Hierarchy entry "Normal data"
      Path "/etc/puppetlabs/code/environments/production/data/nodes/prodnode1.company.yaml"
        Original path: "nodes/%{trusted.certname}.yaml"
        Found key: "profile::baseline::svcname" value: "ntpd"
```

</div>