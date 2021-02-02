# Puppet Code Abstraction - Profiles

## Summary

Frequently the implementation of a technology stack in Puppet code requires:

1. Layers of implementation that are not confined to a single Puppet module
2. Site-specific customizations
3. Configuration data that remains unique across application environments/tiers

To meet these needs, Puppet recommends an abstraction layer called a **Profile**
to act as a "wrapper" Puppet module for the site-specific implementation of a
technology stack.

## Expectations

All new users to Puppet should implement Profiles to be aligned
with current Puppet best practices. Existing Puppet users will
find benefit in switching to a Profile-based implementation, however the
migration to Profiles must ensure consistency to eliminate enforcement gaps.

Organizations with multiple Puppet implementations that are looking to
consolidate may experience Profile namespace collisions due to the flattened
nature of the Puppet catalog. In this scenario, per-Profile Puppet modules
may be necessary. The purpose of the Profile remains important however the naming
of each Profile may need to change to accommodate for multiple tenancy.


## Best Practice Details

### Preferred Option

* Profiles shall be contained within a single `profile` Puppet module.

* Profiles shall be named according to the specific technology being modeled.
  in Puppet code (e.g. `profile::wordpress`, `profile::mysql`, `profile::iis`, `profile::java`)

* Unique implementations of a technology stack may have independently namespaced
  Profiles (e.g. `profile::ssh::server`, `profile::ssh::client`) so long as those unique
  implementations are managed independently of one another. If both implementations
  are typically managed together, then a single Profile is sufficient.

* Profiles may be parameterized to provide an API to the implementation of a
  technology stack.

* Resource-style declaration of a profile (e.g. `class { 'profile::ntp': }` )
  should be limited to avoid the possibility of declaring a Profile two
  different ways during catalog compilation.

* Declaring a Profile from within a Profile is permissible so long as the
  Profile declaration is idempotent (i.e. using the `include` function versus
  resource-style declaration).

* Profiles that are only applicable to a specific platform or operating system
  may be namespaced accordingly for easier code management (e.g.
  `profile::windows::iis`, `profile::linux::ntp`, `profile::osx::loginwindow`).

* Profiles applicable to multiple platforms or operating systems may have
  a single, public entry point (e.g. `profile::dns_nameservers`) that can
  conditionally include private sub-namespaces for better code encapsulation
  (e.g. a `profile::dns_nameservers` profile that does an operating system
  check and includes the `profile::dns_nameservers::linux` class on Linux
  nodes and `profile::dns_nameservers::windows` on Windows nodes).

* Profiles may include individual resource declarations so long as they are
  site/business-specific and relevant to the technology stack (e.g. company SSL
  certificates for an Apache website or similar site-specific customizations).
  If the number of individual resources begins to grow, resources may be
  encapsulated in a site-specific sub-namespace to the Profile (e.g.
  `profile::apache::mycorp`) and included in the base Profile, or in
  a site-specific module with a technology-specific sub-namespace (e.g.
  `mycorp::apache`). The preferred class location depends on tolerance to
  access and permissions (i.e. Profiles may need to be more restricted than
  site-specific modules).


### Alternate Options

* Profiles can be broken out into individual modules to accommodate multiple
  tenancy on a Puppet Server, however uniqueness must be maintained (i.e. there
  can only be one `profile` module per Puppet Environment). A prefix shall be
  used (e.g. `profile_apache` or `wrapper_apache`) to designate that a
  module is NOT a component module but is, instead, a Profile. This should only
  be implemented with good cause, however, as workflow overhead is significantly increased.


### Discouraged Options

* Profiles shall not be named after generic server types (e.g.
  `profile::web_server`, `profile::db_server`) because the lack of specificity
  is in direct conflict with "Role" wrapper classes used for classification by
  server type.

* Profiles shall not contain resources from Component Modules. Only resources
  containing site-specific data shall be declared inside Profiles to preserve
  modularity.


## Feedback / Ideas for Improvement

* Loosely following RFC2119 for wording of things like 'MUST', 'SHALL', and the
  like: https://www.ietf.org/rfc/rfc2119.txt
* Feedback can be provided as an issue on this [Git
  repository](https://github.com/puppetlabs/best-practices/issues).

## Other Information

* https://puppet.com/docs/pe/latest/the_roles_and_profiles_method.html
