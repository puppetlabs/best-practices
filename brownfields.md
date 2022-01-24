# ⚠ WARNING ⚠

This repository is no longer mantained since ~ 2019. While some of these documentation might still be useful, much of it is obsolete or out of date.

For a maintained collection of similar information, please see [Patterns and Tactics](https://puppet.com/docs/patterns-and-tactics).
# Managing Brownfields

## Summary

This best practice describes how customers can use PE to manage brownfields
environments without causing massive amounts of change at once across their
environments.

## Expectations

Most new Puppet users that manage brownfields environments will benefit from
this best practice and most likely discard this pattern once their brownfields
environment is fully compliant.

Many current Puppet users who have had PE in place for some time will not need
this standard, unless they are given further brownfields environments to manage.

## Best Practice Details

### Preferred Option

Thomas Linkin's [noop module](https://forge.puppet.com/trlinkin/noop) provides a
function that can put selected scopes in `noop` mode. The use of this function
at the Profile layer ensures the Profile and its scope are in `noop` mode. A
parameter on the Profile can be used to determine if the Profile is in `noop` or
not. The value for the parameter should be retrieved from Hiera (via traditional
backend or the Data section of the PE Console).

The following is an example of a Profile that uses this design pattern:

```puppet
class profile::monitoring (
  Boolean $class_level_noop = false,
) {

  if $class_level_noop {
    noop(true)
  }
  ...
}
```

This design pattern should only be used in leaf Profiles (e.g. Profiles that do
not include other Profiles) to prevent scope issues with `noop`.

The `class_level_noop` parameter can be set in different levels of the
hierarchy of Hiera. This can be very broad (whole environment) or very granular
(at the node level).

With this approach the Puppet user can determine which Profiles on which nodes
or groups of node should be run in `full enforcement` mode or `noop` mode.

WARNING: the `noop` function overrides `noop` at the command line and in
`puppet.conf`.

WARNING: Using this pattern with a large node count (>10,000) can cause PuppetDB
performance issues.

### Alternate Options

If the customer needs to identify certain nodes as brownfields an external or
custom fact can be also used to differentiate the environments:

```puppet
class profile::monitoring (
  Boolean $class_level_noop = false,
) {

  if $facts['brownfields'] and $class_level_noop {
    noop(true)
  }
  ...
}
```

In the above example the `brownfields` fact is being used to determine if a node
is brownfields and is used in conjunction with the `class_level_noop`
parameter.

### Discouraged Options

* Hardcoding of the function to be either `true` or `false` is discouraged as it
  makes the code very opinionated and will require code changes if circumstances
change.
* Using the `noop` function in Profiles that include other Profiles, which can
  lead to scope issues with `noop`.

## Feedback / Ideas for Improvement

* Loosely following RFC2119 for wording of things like 'MUST', 'SHALL', and the like: https://www.ietf.org/rfc/rfc2119.txt
* Feedback can be provided as an issue on this [Git repository](https://github.com/puppetlabs/best-practices/issues).

## Other Information

* https://forge.puppet.com/trlinkin/noop
* https://puppet.com/docs/puppet/latest/lang_scope.html
* https://puppet.com/docs/pe/latest/managing_nodes/designing_system_configs_roles_and_profiles.html
