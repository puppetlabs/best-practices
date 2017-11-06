# When to use Puppet Tasks vs Puppet DSL Code/Modules

## Introduction

The inclusion of tasks in Puppet Enterprise (PE) represents a paradigm shift in the puppet ecosystem. Prior to Puppet Tasks custom code written in anything other then puppet (or in rare cases ruby) was discouraged.

In this document we, the Puppet Professional Services organization, will present some general guidelines we will be following about when and how tasks should be implemented during an engagement. We hope that these guidelines may be useful to you as you work with tasks and Puppet DSL code in your own environment. 

## Summary

* Use tasks where Puppet DSL code is not suitable because longer term state is not being managed

* Use the most ubiquitous language for your environment and users (e.g. bash on Linux, powershell on Windows)

* Take advantage of existing puppet tools where possible rather than reimplementing in your task 

## Best Practice Details

### Preferred Option

The following is targeted towards tasks shipping in Puppet Enterprise, not
remote [bolt](https://puppet.com/products/puppet-bolt) tasks. This is due to
the inherent complexity of boot-strapping dependencies without a configuration
management platform installed. 

* Tasks should be written in the simplest available language on a given platform. (e.g. bash on Linux or powershell on Windows )

* Tasks should use `puppet resource` and `facter` as much as possible to minimize reimplementation of  platform independent logic.

* Tasks should not install their own prerequisites, puppet code should written for dependencies (e.g. `gem`, `pip` ).

* Tasks should be parametrized rather then have hard-coded values in them.


### Alternate Options

* Tasks may be written in an object oriented interpreted language when libraries
  exist that will increase the capabilities and minimize the code compared to
  simpler languages on the platform ( e.g. `ruby` or `python` ).

* Tasks may use a platform specific language that is the most familiar to broadest user group in your environment. 

* Tasks may use puppets ruby stack when users are agnostic about the language. 
  
* Tasks may use `puppet apply` as the [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) or in a [HERE doc](http://tldp.org/LDP/abs/html/here-docs.html) when site specific parameters must be passed as environmental variables. 

### Discouraged Options

* Tasks shall not be used for ad-hoc configuration management. The exception
  would be when a configuration option is changed before a transition state. An
  example would changing a configuration file, stopping a service, then
  reverting the configuration change after restarting a service.

* Tasks shall not manage a state that is already managed by puppet code on
  the system. 
	* Exceptions would be if puppet runs are incorporated into the task plan and puppet is expected to undo an ad-hoc configuration change as part of the automation timeline (e.g. starting a service after a db migration).

* Tasks shall not implement their own logging framework or host filtering
  logic.

* Tasks shall not download additional non-version controlled code and execute
  it. The obvious exception is the PE Installer if tasks are used for bootstrapping new masters. 
	* A related best practice is that where possible apps should be available via package management

* Tasks shall not execute multiple `puppet apply` commands in a single task.
  Plans should be created with multiple tasks or `puppet resource` should be
  used in simple transitional state changes.

