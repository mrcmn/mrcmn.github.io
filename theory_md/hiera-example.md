<div class="tocoutline">

### Table of Contents

<div class="toc">

- [Example - Apache Configuration](#example---apache-configuration)
- [Using Hiera in existing Puppet Code](#using-hiera-in-existing-puppet-code)
- [Hiera Data hierarchy](#hiera-data-hierarchy)
- [Adding Hiera Data](#adding-hiera-data)
  - [Applying configuration](#applying-configuration)
  - [Example Node 1](#example-node-1-1)
  - [Example Node 2](#example-node-2-1)
- [Review](#review)

</div>

</div>

# Example - Apache Configuration
In this example, we'll walk through the process of removing hard coded data values from puppet code and moving them to hiera. We'll demonstrate how data can be consumed via multiple hiera data layers by adapting hiera data to accomdate for two nodes, running two different operating systems - RHEL and CentOS.

Each node will have the same puppet manifest applied however, based on the system information of the node itself, each of them will recieve slightly different configuration values.

# Using Hiera in existing Puppet Code

We have a simple apache configuration below but all of the values are hardcoded within the class. We want to make those values dynamic by assigning them to Hiera attributes. Below you can find our configuration with hardcoded values:

apache_web.pp


```puppet
class profile::apache_web {
  class { 'apache':
    default_vhost => false,
  }
  apache::vhost { 'vhost.example.com':
    port    => '80',
    docroot => '/var/www/vhost',
  }
}
```

<div class="noninteractive">

```
control-repo/
└─ site-modules/
   └─ profile/
      └─ manifests/
         └─ apache_web.pp
```

</div>


If we want to separate data from code for the **apache_web** profile below, we can do it like this:

apache_web.pp

```puppet
class profile::apache_web(
  String $doc_root,
  String $vhost_name,
  Integer $port,
  Boolean $default_vhost
) {

  class { 'apache':
    default_vhost => $default_vhost,
  }
  apache::vhost { '$vhost_name':
    port    => $port,
    docroot => '$doc_root',
  }
}
```

Note that the `$default_vhost, $vhost_name, $port `and` $doc_root `parameters are now variables which will be automatically populated from Hiera. The variables have not been given default values in this case, which means that you’d be required to set at least default values in your `common.yaml` in Hiera to prevent this class from failing. If you want to be able to use a class with overridable parameters even when no parameters have been explicitly defined in Hiera, you should either define sane defaults for the variables in the code directly, or configure a <a href="https://puppet.com/docs/puppet/latest/hiera_intro.html#hiera_config_layers-module-layer" target="_blank">module-specific Hiera</a> layer to retrieve defaults from there if the normal Hiera hierarchy does not deliver any results. 

In this trivial example, we’ll leverage the **OS** fact to determine if a target node matches a given OS (CentOS or RedHat), it then receives a value specific to that OS. We'll also assign a vhost name based on a specific node using the **certname** fact. The rest of the Hiera values (`default_vhost `and` doc_root) `are defaults for all nodes in this example and will be consumed from the `common.yaml `file/common layer.

# Hiera Data hierarchy

Take a look at the `hiera.yaml `file below and review the data hierarchy under the `paths `key:


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


As you can see, we have 3 Hiera data layers:

1. `nodes/%{trusted.certname}.yaml`
2. `os/%{facts.operatingsystem}.yaml` 
3. `common.yaml`

This means that relevant hiera values in the nodes layer will be accepted first, then the os layer, then common.yaml 


# Adding Hiera Data
Our hiera data will be directly linked to the system attributes(Facts) of our target nodes. The two key system information facts that will determine the configuration for our nodes is operating system and node/certname.


The two nodes we want to target with this configuration have the following system attributes:

### Example Node 1
- Operating system: CentOS
- Certname: `mynodecertname1.company`

### Example Node 2

- Operating system: RedHat
- Certname: `mynodecertname2.company`


Below you can find the layout of our Hiera data directory where our Hiera variables are stored.

```
control-repo/
└─ data/
   ├─ common.yaml
   ├─ os/
   │   └─ CentOS.yaml
   │   └─ RedHat.yaml
   └─ nodes/
       └─ mynodecertname1.company.yaml
       └─ mynodecertname1.company.yaml
```

When these Hiera attribute values are resolved during catalog compilation, they help to form a “complete” configuration, with **code** in the `profile::apache_web` consuming **data** from Hiera.

## Applying configuration<a href="#applying-configuration" aria-hidden="true"></a>

If we now push this configuration to source control and apply the `profile::apache_web` to two different nodes with a slightly different architecture, we'll get the following results:

## Example Node 1
- Operating system: CentOS
- Certname: `mynodecertname1.company`

Below you can find the hiera data that's relevant to this node, based on it's system information:

#### Relevant Hiera data:

common.yaml
```yaml
profile::apache_web::default_vhost: false
profile::apache_web::doc_root: /var/www/vhost
```

data/os/CentOS.yaml
```yaml
profile::apache_web::port: 80
```

data/nodes/mynodecertname1.company.yaml
```yaml
profile::apache_web::vhost_name: mynodecertname1.company.com
```

Based on these hiera values, this node will receive a web server configuration with the following values, once Hiera lookups are reconciled: 

#### Resolved values for mynodecertname1.company

```puppet
class profile::apache_web {
  class { 'apache':
    default_vhost => false,
  }
  apache::vhost { 'mynodecertname1.company.com':
    port    => 80,
    docroot => '/var/www/vhost',
  }
}
```

## Example Node 2

- Operating system: RedHat
- Certname: `mynodecertname2.company`

Below you can find the hiera data that's relevant to this node, based on it's system information:

#### Relevant Hiera data:

common.yaml
```yaml
profile::apache_web::default_vhost: false
profile::apache_web::doc_root: /var/www/vhost
```

data/os/RedHat.yaml
```yaml
profile::apache_web::port: 8081
```

data/nodes/mynodecertname2.company.yaml
```yaml
profile::apache_web::vhost_name: mynodecertname2.company.com
```

#### Resolved values for mynodecertname2.company


```puppet
class profile::apache_web {
  class { 'apache':
    default_vhost => false,
  }
  apache::vhost { 'mynodecertname2.company.com':
    port    => 8081,
    docroot => '/var/www/vhost',
  }
}
```


# Review
Both configurations receive the same value for `default_vhost` and `doc_root` as they are defined within the `common.yaml `data layer which is common to all nodes.

Each node receives different Hiera values for `vhost_name` which is specific to them at a node level, identified by their PE certname in their respective yaml files (`mynodecertname1.company.yaml` and `mynodecertname2.company.yaml`) in the nodes data layer. 

They also each receive different values for the `port` attribute based on the type of OS they’re running, RedHat or CentOS with values coming from `RedHat.yaml` and `CentOS.yaml` files in the OS data layer
