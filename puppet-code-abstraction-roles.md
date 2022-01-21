# Puppet Code Abstraction - Roles

## Summary

In a well-encapsulated Puppet deployment, there exist:

* Component Modules that contain Puppet code and plugins for managing the
  many configuration permutations of a particular piece of technology
* Hiera configuration data to account for the differences in the implementation
  of a piece of technology between different environments, business units, or
  other logical business-level separation
* "Profile" wrapper-classes that provide a single Puppet class to configure all
  the components of a layered technology stack

The final piece missing is a "Role", or a wrapper Puppet class to assist in the
classification of Profiles to a node based on the machine "type."  Roles may
contain conditional logic to match a node with the proper Profile(s) based on
information such as operating system, business unit, and any other necessary
node-specific data.


## Expectations

All new users to Puppet should implement Roles to be aligned
with current Puppet best practices. Existing Puppet users will
find benefit in switching to a Role-based implementation, however the
migration to Roles must ensure consistency to eliminate enforcement gaps.

Organizations with multiple Puppet implementations that are looking to
consolidate may experience Role namespace collisions due to the flattened
nature of the Puppet catalog. In this scenario, per-Role Puppet modules
may be necessary. The purpose of the Role remains important however the naming
of each Role may need to change to accommodate for multiple tenancy.


## Best Practice Details

### Preferred Option


* The role module should be versioned as part of the control-repo to provide
  the best balance between uniformity and ease of workflow.

* Roles shall be limited to one per node where possible.

* Roles shall be named generically according to machine "type" (e.g.
  `role::application_server`, `role::database_server`,
  `role::middleware_host`).

* Roles may include the parent namespace for a more layered approach to
  declaring Profiles (e.g. `role::app_server` may include the base `role`
  class, and `role::app_server::pci` will include the `role::app_server`
  parent class).

* Hiera lookups may be included in Roles only if the lookup aids in the declaration
  of Profiles.


### Alternate Options

* The role module may alternatively be maintained as an independently versioned module.

* Roles may use inheritance to ensure that sub-classes of the `role` class
  will inherit from the parent namespace (e.g. the base `role` class may
  include all Profiles for ALL nodes in the organization, the
  `role::application_server` child namespace then inherits from `role` and
  extends upon the base `role` class by including additional Profiles required
  by Application Servers, and `role::application_server::pci` then inherits and
  extends upon the `role::application_server` class with Profiles to be included
  on all Application Servers that must adhere to PCI specifications). This is
  an alternate option because the use of inheritance is generally discouraged
  within the Puppet DSL.  This is an alternate option to avoid surprise.

* Roles may contain business logic to determine which Profiles shall be declared
  for a given role and aid with classification. For example, the
  `role::application_server` Role may check the `$::os['family']` fact and
  include `profile::iis` on Windows nodes, `profile::jboss` on Linux nodes in
  the San Francisco datacenter, and `profile::weblogic` on Linux nodes in the
  North Carolina datacenter. This allows all Application Servers to be
  classified with the same Role regardless of the technology stack being
  implemented.

* Roles can be broken out into individual modules to accommodate multiple
  tenancy on a Puppet Server, however uniqueness must be maintained (i.e. there
  can only be one `role` module per Puppet Environment). A prefix shall be
  used (e.g. `role_application_server`) to designate that a
  module is NOT a component module but is, instead, a Role. This should only
  be implemented with good cause, however, as workflow overhead is significantly increased.

* In environments where node uniqueness is quite high (e.g. self-serve portals that
  spin up nodes ad-hoc), classifying nodes with more than one role is acceptable.


### Discouraged Options

* Roles shall not be named after specific technologies (e.g. `role::tomcat`,
  `role::jboss`) because the specificity is in direct conflict with the purpose
  of "Profile" wrapper classes.

* Roles shall not contain resources - only declarations of Profiles and conditional
  logic.

## Feedback / Ideas for Improvement

* Loosely following RFC2119 for wording of things like 'MUST', 'SHALL', and the
  like: https://www.ietf.org/rfc/rfc2119.txt
* Feedback can be provided as an issue on this [Git
  repository](https://github.com/puppetlabs/best-practices/issues).

## Other Information

* https://puppet.com/docs/pe/latest/the_roles_and_profiles_method.html
