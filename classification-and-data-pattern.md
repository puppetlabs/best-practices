# ⚠ WARNING ⚠

This repository is no longer mantained since ~ 2019. While some of these documentation might still be useful, much of it is obsolete or out of date.

For a maintained collection of similar information, please see [Patterns and Tactics](https://www.puppet.com/docs/patterns-and-tactics/latest/patterns-and-tactics.html).
# Classification and Data Pattern Standards

## Summary

This best practice defines the preferred path for how to classify in the PE GUI,
and the preferred place to store data. In summary the console classification
should use 1 role per group, and data should be stored in Hiera to avoid split
sources of truth. PE configuration is the exception which is currently done in
the console (but likely to change in the future).

## Expectations

All new Puppet Enterprise users should use the below node classification and
data standards.

This best practice may not fit with high complexity environments with many
snowflake nodes.

## Best Practice Details

### Preferred Option

#### Classification via the PE classifier

The best practice for classification is for each node to use one role/Node
Group.

There are two reasons for this best practice:

1. Users should not use parameters in the PE console as the PE classifier is not
   versionable
2. Using profiles in Node Groups (node group as a role) reduces the ability to
   test adding new profiles to a role

#### Data

The recommended best practice is to use params in Hiera, except for PE
infrastructure, which should be set as parameters in the PE Classifier

This best practice, in general, does not recommend users editing parameters in
the console. However, Puppet Enterprise documentation advises users to edit
parameter values for PE Infrastructure in the console. Therefore, for user
modules, the best practice is that all data should be in Hiera. For PE Infra,
the best practice is that all data should be in the PE Console as parameters.

The PE console and Hiera have no visibility of each other, so err on Hiera for
most things that need a data override.

There should be one source of truth for data, since PE itself puts some
parameters for PE infra in the PE console all other data for PE Infra should go
into the PE console instead of splitting some of it in Hiera and some in the
console.

#### Per Node classification

Treat per node classification just like all other classification - one role in a
node group.

### Alternate Options

These options may be used if the situation requires.

#### Classification via the PE Classifier

In situations where a provisioning system is used, or where there are teams that
are solely given access to the PE Classifier to change params it may be
necessary to expose those params in the console. The Puppet administrators may
require this if end users have limited Puppet experience or limited experience
with Git etc.  In this situation the node is still classified with one role.

Parameters should be exposed that an end user (whoever that may be) can modify
for whatever reason: this may be different in prod to non-prod.

This should be avoided unless required as the PE classifier cannot be versioned
and this creates a second source of truth which may make debugging more
difficult.

### Discouraged Options

#### Classification - Multiple Profiles per Node Group when used with "have it your way" provisioning systems

In very rare situations an installation may have a provisioning system where the
users are able to compose the configuration of a node by picking which profiles
that node will be classified with. Essentially making them able to provision
bespoke server configurations.

This should be avoided at all costs unless absolutely required as it can lead to
a proliferation of node groups. Additionally it makes doing RSpec or acceptance
testing at the role level very difficult (how to test if someone is overriding
the standard operating environment(SOE) as an example)

## Feedback / Ideas for Improvement

* Loosely following RFC2119 for wording of things like 'MUST', 'SHALL', and the
  like: https://www.ietf.org/rfc/rfc2119.txt
* Feedback can be provided as an issue on this [Git
  repository](https://github.com/puppetlabs/best-practices/issues).

## Other Information

* https://puppet.com/docs/pe/latest/managing_nodes/grouping_and_classifying_nodes.html
* https://puppet.com/docs/puppet/latest/hiera_quick.html
* https://puppet.com/docs/pe/latest/managing_nodes/the_roles_and_profiles_method.html
