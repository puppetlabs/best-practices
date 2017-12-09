# Puppet Modules and the Control Repo

## Summary

Puppet users requiring code consistency use a tool like Puppet Enterprise's
Code Manager or R10k to ensure that the required Puppet environments containing all
necessary modules (at their requested versions) are present on all of their
Puppet masters. The decision of where to store Puppet modules (whether in the
control repo or in their own version control repositories) quickly arises for
both tools. This document defines the standard that can be referenced to assist
in making that decision.

## Expectations

* All Puppet customers using Code Manager or R10k will benefit from the
  guidelines in this standard.


## Standard Details

### Preferred Option

* The modules used for the Roles and Profiles design pattern should be stored
  in the control repo unless there is a good reason NOT to do so, such as:
    - If separate teams manage the Puppet infrastructure and develop Puppet
      code
    - The organization requires separate Puppet modules and version control
      repositories for each individual profile
    - Other security or organizational requirements

* Simple component modules (one or two classes and a handful of resources) can
  be stored within the control repo provided one of the following is true:
    - It's anticipated that the module won't need any/many changes
    - Creating a new repository for the module would be excessive
    - Sharing history/access with the control repo is acceptable
    - Knowledge/experience with version control is limited

* All other component modules should be stored in their own version control repository
  and listed within `Puppetfile`

* All modules listed in `Puppetfile` for the "production" environment should
  have a version specified and "pinned" to a commit/tag/release to eliminate the
  possibility of unexepected "upgrades" during a code deployment


### Alternate Options

* The modules used for the Roles and Profiles design pattern may be stored
  within their own version control repository should it be deemed better for
  the organization

### Discouraged Options

* Customers still using a "monolithic module repository" (i.e. a version
  control repository containing all modules necessary for a Puppet environment)
  should transition modules to their own repositories or to being stored within
  the control repo according to this standard


## Feedback / Ideas for Improvement

* Loosely following RFC2119 for wording of things like 'MUST', 'SHALL', and the like: https://www.ietf.org/rfc/rfc2119.txt


## Other Relevant Information


