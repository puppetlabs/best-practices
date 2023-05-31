# ⚠ WARNING ⚠

This repository is no longer mantained since ~ 2019. While some of these documentation might still be useful, much of it is obsolete or out of date.

For a maintained collection of similar information, please see [Patterns and Tactics](https://www.puppet.com/docs/patterns-and-tactics/latest/patterns-and-tactics.html).
# Use of %{environment} in hiera hierarchy

# Summary

The `%{environment}` variable SHOULD NOT be used in the definition of a Hiera
hierarchy level. If a Hiera hierarchy level is needed pertaining to deployment
tier (Dev, Test, Prod), a custom fact called `deployment_tier` MAY be created
and used in defining hierarchy level(s).

# Expectations

New customers working with Puppet Customer Success should be directed to avoid
using `%{environment}` in their Hiera hierarchy.

Existing customers using `%{environment}` in their Hiera hierarchy should be
informed of the problem and guided towards using a custom fact.

# Best Practice Details

## Definitions

In Puppet today, an environment is:

* A directory on the master containing Puppet code
* The value of `%{environment}`

To most users, an environment is:

* A set of computer systems in which software is deployed and executed
* A stage in the software development lifecycle pipeline
* Dev, Test, Prod, etc.

See
[https://en.wikipedia.org/wiki/Deployment\_environment](https://en.wikipedia.org/wiki/Deployment_environment)

## Workflow

In Puppet, different directories—different Puppet environments—contain different
versions of code. To use a different version of Puppet code, even temporarily, a
node must be configured to use a different Puppet environment.

Proposed changes to infrastructure are tested and deployed first to a deployment
environment (Dev, Test) in an early stage of the pipeline. If the changes are
successful, they will be deployed to environments further along the pipeline,
until eventually being deployed to systems in the production environment.

In development and testing of Puppet code it is common to create a new version
of code and canary test it before deploying it to the whole deployment
environment. This is done by creating a new short-lived Puppet environment, and
configuring one or more nodes in a deployment environment such as Dev or Test to
temporarily use code from the short-lived Puppet environment.

## Problem Statement

During canary test runs the value of `%{environment}` on canary nodes will
reflect the name of the temporary Puppet environment, not their deployment
environment. The nodes will remain conceptually part of the Dev or Test
deployment environment, but their `%{environment}` variable will no longer match
Dev or Test.

Data in Hiera is often logically organized to include a hierarchy level
pertaining to the deployment environment. Different service IPs are used in
Production than are used in Test. Different account passwords are deployed to
Test than to Dev. And so on.

A node in the Dev deployment environment should always use the Dev root
password, regardless of what version of Puppet code it is currently running.

In standard Puppet workflows `%{environment}` is NOT a reliable indicator of
what deployment environment a node belongs to. `%{environment}` is more reliably
an indicator of what version of Puppet code a node is running. Data is typically
defined pertaining to a user's deployment environments, not to Puppet code
versions.

If a Hiera hierarchy contains `%{environment}`, canary nodes being tested
against a temporary Puppet environment will effectively lose access to data
pertaining to them in their deployment environment. The specific effect of this
happening will vary depending on what other data the node has available to fall
back to in other tiers of the hierarchy, but it will be in general unexpected
and undesirable.

### Example of the problem

Let's assume you need to set your ntp server ip address differently per what a
user calls an environment ( and we call a deployment_tier ).  In dev we want
10.10.25.25 and in production we want 10.10.29.29.  We set this by making an
environment level of our hiera hierarchy and making a file for each environment.

hiera.yaml

```
hierarchy:
  - %environment
  - common
```

```
cat dev.yaml
---
ntp::server: 10.10.25.25
```

```
cat production.yaml
---
ntp::server: 10.10.29.29
```

Now we'd like to make a code change and we create a feature branch of our
control-repo to test the change.  We make a feature branch called my_feat_branch
and it gets deployed on the master as a Puppet environment of the same name.

When we go to test the change we get an unexpected result in that our ntp
servers in ntp.conf are set to the defaults of the module instead of what we
wanted which is the ntp server for the dev user environemnt ( application_tier
).  This is because we did not create a my_feat_branch.yaml in the hiera datadir
to set the ip address of our ntp server.

In this simple example you could imagine just copying the dev.yaml file to
my_feat_branch.yaml to get it to work but this would mean your adding files to
your data directory simply for testing purposes instead of for the final code
/data.

The resolution is to simply separate Puppet Environments from data you want for
user environments ( application_tiers ).  When you make a custom fact for
application tier when you go to test code you can supply whatever
application_tier you'd like for testing without worrying about the name of the
Puppet Environment.

## Preferred Option

A custom fact called `deployment_tier` MAY be implemented which reflects which
deployment tier (Dev, Test, Prod) a node belongs to.

The custom fact may be defined as a trusted fact before install, or defined via
an external fact pre or post-install.

If a Hiera hierarchy level is needed pertaining to deployment tier, the
`%{deployment_tier}` value MAY be used in defining the hierarchy level(s). The
`%{environment}` value SHOULD NOT be used in the definition of a Hiera hierarchy
level.

Configuration of the Hiera hierarchy should not be confused with configuration
of the Hiera datadir. `%{environment}` can and should be used to configure the
Hiera datadir. This is already [the default datadir
configuration](https://puppet.com/docs/puppet/latest/hiera_config_yaml_5.html#the-default-configuration).

## Alternate Options

Users may prefer to use a custom fact named something other than
deployment_tier. This name is preferred as it does not re-use the term
"environment", is generic, yet is indicative of the low-level workflow it
pertains to.

The exact name has no impact on the technical details of the solution. Other
fact names discussed in Gary's blog, and thus likely to be recognizable to other
users, include `app_tier` or `application_tier`.

## Discouraged Options

N/A

## Feedback / Ideas for Improvement

* Loosely following RFC2119 for wording of things like 'MUST', 'SHALL', and the
  like: https://www.ietf.org/rfc/rfc2119.txt
* Feedback can be provided as an issue on this [Git
  repository](https://github.com/puppetlabs/best-practices/issues).

# Other Information

Gary Larizza on the problem with `%{environment}`:
http://garylarizza.com/blog/2014/10/24/puppet-workflows-4-using-hiera-in-anger/
