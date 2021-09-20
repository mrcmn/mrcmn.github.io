<div class="tocoutline">

### Table of Contents

<div class="toc">

- [Overview](#overview)
- [Why use it?](#why-use-it)
- [How does it work?](#how-does-it-work)
  - [What types of activity can be controlled?](#what-types-of-activity-can-be-controlled)
  - [Creating Users](#creating-users)
    - [Local](#local)
    - [External Directory Service](#external-directory-service)
  - [Creating Roles](#creating-roles)
    - [Adding users to Roles](#adding-users-to-roles)
    - [Adding Permissions to Roles](#adding-permissions-to-roles)
  - [Summary](#summary)

</div>

</div>

# Overview<a href="#overview" aria-hidden="true"></a>

Role based access control (RBAC) is used to grant users or groups access to Puppet Enterprise to perform specific actions. 

# Why use it?<a href="#why-use-it" aria-hidden="true"></a>

RBAC can allow multiple users or groups to use the same platform to safely achieve different outcomes. Visibility and control can be limited to specific actions in line with your security and privacy requirements. RBAC also enables teams to create guard rails to safely allow “consumers of automation” to leverage Puppet for self service capabilities.

# How does it work?<a href="#how-does-it-work" aria-hidden="true"></a>

Assigning permissions in Puppet Enterprise consists of 3 main parts:

* Creating user roles
* Adding users to roles
* Assigning permissions to those roles. 

With RBAC in PE, users are explicitly assigned permissions and they are assigned based on “role”. This means that a user won’t be able to perform any actions in PE until they are added to a role which has predefined permissions.

## What types of activity can be controlled?<a href="#what-types-of-activity-can-be-controlled" aria-hidden="true"></a>

Almost anything you can do within PE: access to the console, view edit and creation of node groups, Task/Plan and Puppet runs, certificate approvals and more. You can find an exhaustive list of available permissions in the <a href="https://puppet.com/docs/pe/latest/rbac_permissions_intro.html#user_permissions" target="_blank">PE docs</a>.

## Creating Users<a href="#creating-users" aria-hidden="true"></a>

You can create “local” users or add existing users and user groups that have been set up in your external directory service. 

### Local<a href="#local" aria-hidden="true"></a>

1. To create a local user in PE, you’ll need to navigate to the **Access Control** under **Admin** in the sidebar in the PE console. 
2. Under the **Users** tab, you can add a user by entering their name under **Full name** and adding a username to the **Login** field.
3. Click **Add local user** to create the user.
2. Once created, click the name of the user and then click **Generate password reset**. You’ll then be presented with a link to send to the user to create a password for themselves.

### External Directory Service<a href="#external-directory-service" aria-hidden="true"></a>

You can integrate with external directory services such as OpenLDAP, Active Directory or a SAML identity provider. You can find more information on how to do that in the  <a href="https://puppet.com/docs/pe/latest/rbac_ldap_intro.html#connecting_puppet_enterprise_with_external_directory_services" target="_blank">Connecting external directory services to PE</a> section of the PE docs.

## Creating Roles<a href="#creating-roles" aria-hidden="true"></a>

1. Navigate to the **Access Control** from the sidebar in the Puppet Enterprise console.
2. Click on the **User Roles** tab. 
3. You’ll find a list of default user roles. To create your own, enter the **Name** and **description** for your role and then click **Add Role**.

### Adding users to Roles<a href="#adding-users-to-roles" aria-hidden="true"></a>

1. Navigate to the **Access Control** from the sidebar in the PE console.
2. Click on the **User Roles** tab. 
3. Click on the role you wish to add users to.
4. From the **Member users** tab and then select the dropdown under **User name** and choose the user you want to add to the role. 
5. Click **Add user**.
6. Click **Commit 1 change** to save changes.

### Adding Permissions to Roles<a href="#adding-permissions-to-roles" aria-hidden="true"></a>

1. Navigate to the **Access Control** from the sidebar in the PE console.
2. Click on the **User Roles** tab. 
3. Click on the role you wish to add permissions to.
4. Click on the **Permissions** tab.
5. Under **Add a permission**, click the dropdown and select the permission that you want to enable for the user role and then click **add**.
6. Once you’ve selected all of your desired permissions, click **Commit &lt;x> changes** to save changes. 

   #### Example
   If you want to enable a role to only allow one task to be run against a specific target node group, you would need to add these permissions:

   | Type              | Action                    | Instance                             | 
   | -----------       | -----------               | -----------                           |
   | Console           | View                      |                                       |
   | Job orchestrator  | Start, stop and view jobs |                                       |
   | Node Groups       | View                      | _(your_node_group)_                   |
   | Tasks             | Run Tasks        | _(your_task_name)_ Permitted Nodes:  _(your_node_group)_ | 

    Now when the user logs in to the PE console using an account which is part of a role with specific permissions, they’ll only be able to perform those actions specified in the role and nothing more.

## Summary<a href="#summary" aria-hidden="true"></a>

RBAC in PE can be a great tool to allow other teams or junior members of the team to participate in automation with guard rails. It can also be useful to open up possibilities around self service automation to allow “consumers” of automation to leverage PE to achieve automation tasks without requiring significant Puppet skills or knowledge of the Puppet platform.
