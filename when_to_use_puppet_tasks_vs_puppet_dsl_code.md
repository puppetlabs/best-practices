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

**The following is targeted towards tasks shipping in Puppet Enterprise, not
remote [bolt](https://puppet.com/products/puppet-bolt) CLI tasks.** This is due to
the inherent complexity of boot-strapping dependencies without a configuration
management platform installed. 

* Tasks should be written in the simplest available language on a given platform. (e.g. bash on Linux or powershell on Windows )

* Tasks should use `puppet resource` and `facter` as much as possible to minimize reimplementation of  platform independent logic.

* Tasks should not install their own prerequisites, puppet code should be written for dependencies (e.g. `gem`, `pip` ). Tasks should focus on execution of just the action desired. Pre-requisites should be considered long term state and thus managed with puppet code.

* Tasks should be parametrized rather then have hard-coded values in them.


### Alternate Options

* Tasks may be written in an object oriented interpreted language when libraries
  exist that will increase the capabilities and minimize the code compared to
  simpler languages on the platform ( e.g. `ruby` or `python` ).

* Tasks may use a platform specific language that is the most familiar to broadest user group in your organization. 

* Tasks may use puppets ruby stack when users are agnostic about the language. Puppet ruby is a known common denominator and reduces the need for management of ruby as a pre-requisite.
  
* Tasks may be written in the puppet language when no parameters are required; If parameters are required they must be **TBD from tech discuss conversation**

### Discouraged Options

* Tasks shall not be used for ad-hoc configuration management. The exception
  would be when a configuration option is changed before a transition state. An
  example would changing a configuration file, stopping a service, then
  reverting the configuration change after restarting a service.

* Tasks should not manage a state that is already managed by puppet code on
  the system. 
	* Exceptions would be if puppet runs are incorporated into the task plan(Not yet a feature in PE tasks) and puppet is expected to undo an ad-hoc configuration change as part of the automation timeline (e.g. starting a service after a db migration).

* Tasks should not reimplement existing functionality of puppet or the puppet tasks framework. Feedback from the task shall always make it's way back to the Puppet Enterprise console. Ex. Don't implement a different logging framework or host filtering logic within a task rather than using PQL.
	
* When downloading scripts or other content, the URL or source should specify a version. E.g. Source it from a tag rather than a branch. This will ensure you get the expected result when the task is run.

* Tasks shall not execute multiple `puppet apply` commands in a single task.
  Plans should be created with multiple tasks or `puppet resource` should be
  used in simple transitional state changes.

