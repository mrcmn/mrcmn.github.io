<div class="tocoutline">

### Table of Contents

<div class="toc">

- [Example: Web server Role](#example-web-server-role)
- [Profiles](#profiles)
  - [Apache profile](#apache-profile)
  - [Firewall profile](#firewall-profile)
- [Role](#role)
  - [Web server Role](#web-server-role)
- [Applying configuration](#applying-configuration)

</div>

</div>

# Example: Web server Role<a href="#example-web-server-role" aria-hidden="true"></a>

In this example, we’ll walk you through how you can integrate two profiles into a single role. 

One profile will manage firewall configuration and the other will manage the installation and configuration of an apache webserver.

When combined, we'll have a "complete" configuration (role) that we can then easily apply to a group of nodes.

#### Modules used in this example:

- <a href="https://forge.puppet.com/modules/puppetlabs/apache" target="_blank">puppetlabs-apache</a>

- <a href="https://forge.puppet.com/modules/puppetlabs/firewall" target="_blank">puppetlabs-firewall</a>

# Profiles<a href="#profiles" aria-hidden="true"></a>
To start, we’ll need to create two profiles which will form the basis of the role configuration. 

## Apache profile<a href="#apache-profile" aria-hidden="true"></a>

The first profile will consist of a configuration for a basic apache web server. We’ve used some example code from the <a href="https://forge.puppet.com/puppetlabs/apache#beginning-with-apache" target="_blank">documentation for the puppetlabs/apache module</a>.

#### Apache module code example:


```puppet
class { 'apache':
  default_vhost => false,
}
apache::vhost { 'vhost.example.com':
  port    => '80',
  docroot => '/var/www/vhost',
}
```

We can take the code example above change the vhost name and specify a custom doc root. We can then wrap it in a simple profile class and store it in the within `site-modules/profile/manifests` directory in the Control Repo:

apache_web.pp


```puppet
class profile::apache_web {
  class { 'apache':
    default_vhost => false,
  }
  apache::vhost { 'mynode.company.com':
    port    => '80',
    docroot => '/var/www/mynode.company.com',
  }
}
```


**<span style="text-decoration:underline;">Control Repo location:</span>**

<div class="noninteractive">

```
control-repo/
└─ site-modules/
   └─ profile/
      └─ manifests/
         └─ apache_web.pp
```

</div>

## Firewall profile<a href="#firewall-profile" aria-hidden="true"></a>

The second profile will consist of a basic firewall configuration to allow http and https access. We’ve used an example from the <a href="https://forge.puppet.com/modules/puppetlabs/firewall#application-specific-rules" target="_blank">documentation for the puppetlabs/firewall module</a>.

### Firewall module code example:

```puppet
 firewall { '100 allow http and https access':
    dport  => [80, 443],
    proto  => 'tcp',
    action => 'accept',
  }
```

Just as before, we can take the code example above and wrap it in a simple profile class and store it in the within `site-modules/profile/manifests` directory in the Control Repo:

firewall.pp


```puppet
class profile::firewall {
 firewall { '100 allow http and https access':
    dport  => [80, 443],
    proto  => 'tcp',
    action => 'accept',
  }
}
```

**<span style="text-decoration:underline;">Control Repo location:</span>**

<div class="noninteractive">

```
control-repo/
└─ site-modules/
   └─ profile/
      └─ manifests/
         └─ apache_web.pp
         └─ firewall.pp
```

</div>

Now we have two profiles, `profile::apache_web` and `profile::firewall` that we can use as units of automation, like lego bricks.


# Role<a href="#role" aria-hidden="true"></a>

We now want to create a role that combines both profiles or “lego bricks”, so that we can assign this role to a server.

## Web Server Role<a href="#web-server-role" aria-hidden="true"></a>

To create a role, we’ll declare our two profile classes within a role manifest, and store it within `site-modules/role/manifests` directory in the Control Repo like so:

web_server.pp

```puppet
class role::web_server {
  include profile::firewall
  include profile::apache_web
}
```


**<span style="text-decoration:underline;">Control Repo location:</span>**

<div class="noninteractive">

```
control-repo/
└─ site-modules/
   ├─ role/
   │   └─ manifests/
   │      └─ web_server.pp
   └─ profile/
      └─ manifests/
         └─ apache_web.pp
         └─ firewall.pp
```

</div>

Now we have a single role that we can assign to a group of servers in the Puppet Enterprise user interface - `role::web_server`.

# Applying configuration<a href="#applying-configuration" aria-hidden="true"></a>

Once you’ve pushed your changes to your Control Repo, it should be available on the Puppet Server within a few seconds. This will happen automatically if the Puppet Server is set to sync with source control each time there’s a commit to the control repo, alternatively, you’ll need to run `puppet-code deploy --all --wait` directly on the Puppet Server to pull the newest version of the Control Repo.

You can then navigate to **Node groups** and click on your target node group. Navigate to the **Classes** tab.

Type the name of the class under **Add new class**, choose your role (`role::web_server`) and then click **Add class** then **Commit 1 change**.

> If your class doesn’t appear, you may need to click **Refresh** to pick up the latest class definitions.

From your node group, in the top right corner, click **Run > Puppet** then **Run job**.
