# ⚠ WARNING ⚠

This repository is no longer mantained since ~ 2019. While some of these documentation might still be useful, much of it is obsolete or out of date.

For a maintained collection of similar information, please see [Patterns and Tactics](https://www.puppet.com/docs/patterns-and-tactics/latest/patterns-and-tactics.html).

You should prefer setting PE settings in user hiera data unless you have reason not to set them there.

Reasons to set settings in user hiera data: 
 - Can configure the hierarchy to suit the needs of the user environment. 
 - Can use data in the Console for UI for visibility / RBAC
 - Can be easily managed via a control-repo / hiera data repo 

Reasons not to set settings in `pe.conf` include:
 - Not all PE admins have root access on the Master to edit `pe.conf`
 - Admins prefer to save settings in the Console UI for visibility
 - User hiera hierarchies provide appropriate levels for configuring parts of PE that are not afforded by the internal PE hiera configuration  
 - Settings in `pe.conf` can be overridden by user configuration data

Reasons to set settings in `pe.conf`:
 - `pe.conf` is the only part of data that is available on a fresh install of PE.  If you need something right away during install then it should be in `pe.conf`.  Settings in user data will only apply after PE installation is complete and you sync your code / data with code manager.  
 - `pe.conf` should hold the minimum settings you need to successfully bootstrap your install.  

There are 2 ways to configure settings in Puppet Enterprise:

 1. Puppet Enterprise (PE) Configuration Data
  - This is any data set in `/etc/puppetlabs/enterprise/conf.d/`.  The most common file you’ll interact with here is `pe.conf` which is how you installed PE and one of the ways you can change configuration in PE.  You may notice a `nodes/` directory with files named after nodes in your PE infrastructure.  These files are created automatically for you to keep your PE configuration data in sync with your user data.
 2. User Configuration Data
  - This data comes from 2 places.  Your own hiera data, including data you set in the PE console UI, and parameters you set in the PE console UI

It’s important to understand how using any one of these methods results in settings being applied at 2 different times.

 1. When running `puppet agent`
 2. When running `puppet infra configure` ( which is also the primary step in an upgrade or install) 

Let’s go through each way of configuring PE and how it results in settings changing.

When you first install PE, all configuration is sourced from `pe.conf` but after you start using PE then your user configuration data comes into play.  Whenever you run `puppet infra configure` your settings are collected from your user configuration data, then they are merged into PE Configuration Data.  

The normal rules of data in Puppet apply here.  If you set parameters in the console those take precedence over hiera data independent of whether that [hiera data is set in the console](https://puppet.com/docs/pe/2018.1/grouping_and_classifying_nodes.html#task-2210)  or in files on disk.  By default, data in the console takes precedence over files on disk but this is based on your own [configuration of hiera in hiera.yaml](https://puppet.com/docs/pe/2018.1/grouping_and_classifying_nodes.html#task-5039).  

The important thing to note is that if you set a setting in `pe.conf` at install time and then set that setting in your user configuration data when you run `puppet infra configure` then the setting from your user configuration data will take precedence over what was originally set in `pe.conf`.

Settings in PE configuration data are the end result of merging data in the following way.  

 1. Parameters in the console
 2. Your hieradata 
 3. Pe.conf

User configuration data is always preferred over data set in PE configuration data.  If a setting is set in both user and PE configuration data then the user data will take precedence over whatever is already in PE configuration data.
