# Summary

The pattern of managing Hieradata in a separate repository is both common and
acceptable. Doing so provides the ability to decouple the management of configuration
data from that of the Puppet code base. In general, separating the configuration data
from the Control Repository should only be done if a compelling need is identified,
for example:

* Delegating control or access to data
* Multiple Hieradata repositories are used to provide granular access
* Code development occurs at a different cadence than data changes
* Deployment process still dependant on promotion through static tiers

The `Puppetfile` is used to manage this separate repository when following this standard..


# Expectations

All new customers working with someone in CS should be directed to follow this
standard if, and only if, there is an identified need for decoupling as
described above.

This standard may also be applied to existing customers if a need is identified. However
application of this standard to an existing customer may not straight forward. The impact of
adopting this standard for an existing customer's workflow(s) should be explored and considered.


# Standard Details

## Customers on Puppet Enterprise 2016.4 and later

The `Puppetfile` should contain a `mod` entry for the `hieradata` repository. This
entry will appear the same as a Puppet module entry, but will install the contents
of the `hieradata` repository into a separate path. The `:install_path` option is
available in PE version 2016.4 and above.

```ruby
mod 'default',
  :git          => 'git@git.example.com:site_data.git',
  :install_path => 'hieradata'
```

This will install into the environment into a `hieradata` path alongside the `modules`
path. The standard `hiera.yaml` will use this path for its `:datadir`. You will need
to update your `hierarchy` to reflect the name of the hieradata repository.

This method will allow the use of multiple hieradata repositories, for cases in which
multiple teams must own separate pieces of the infrastructure. Note that this is not
recommended unless strictly required.

```yaml
---
:backends:
  - yaml
:hierarchy:
  - "default/nodes/%{::trusted.certname}"
  - default/common

:yaml:
# datadir is empty here, so hiera uses its defaults:
# - /etc/puppetlabs/code/environments/%{environment}/hieradata on *nix
# - %CommonAppData%\PuppetLabs\code\environments\%{environment}\hieradata on Windows
# When specifying a datadir, make sure the directory exists.
  :datadir:
```

## Customers on older versions of Puppet Enterprise

The `Puppetfile` should contain a `mod` entry for the `hieradata` repository. This
entry will resemble that of any Puppet module, and the `hieradata` directory will
be installed into the environment's default `modulepath` location (`modules` directory).

This option should not be used in Puppet Enterprise 2016.4 or above.

```ruby
mod 'hieradata',
  :git => 'git@git.example.com:site_data.git',
```

Since this does not use the standard path for hieradata, you will need to update
your `hiera.yaml` to reflect this:


```yaml
---
:backends:
  - yaml
:hierarchy:
  - "nodes/%{::trusted.certname}"
  - common

:yaml:
# datadir is empty here, so hiera uses its defaults:
# - /etc/puppetlabs/code/environments/%{environment}/hieradata on *nix
# - %CommonAppData%\PuppetLabs\code\environments\%{environment}\hieradata on Windows
# When specifying a datadir, make sure the directory exists.
  :datadir: /etc/puppetlabs/code/environments/%{environment}/modules/hieradata
```

## Deployment Considerations

Choosing to manage your data in the manner described in this standard will require additional attention
be paid to the process of deploying changes. Prior to moving the Hieradata into it's own repository, when it
was stored inside the Control Repository, a change to data would invoke a deployment of the changes if a WebHook
trigger was configured on the Control Repository. Once the Hieradata is stored separately, a change to the data
will not trigger the configured WebHook on the Control Repository. If auto deployment of Hieradata changes is
desired, the Hieradata repository (or repositories) may be configured with a WebHook trigger as well.

If a WebHook trigger is configured on the Hieradata repository, it is important to realize that this trigger will
be invoking Code Manager to perform a deployment of the Control Repository branch named after the Hieradata repository branch that
has changed. This trigger can "fail" to achieve the desired result for a few reasons. First, if the branch name on the Hieradata
repository that triggered Code Manager does not match a branch name on the Control Repository then a deploy will not be invoked.
Second, if version locking of the Hieradata repository is` being done, the latest changes won't deploy until that value is updated.


## Discouraged Options

### Gary Larizza Blog method
In the past, managing hieradata as a separate "source" in `r10k` has been recommended.
Many customers are using instructions from Gary Larizza's (now dated) Puppet Workflow blog
to implement this method.

* http://garylarizza.com/blog/2014/03/07/puppet-workflow-part-3b/

This approach is now discouraged, and is considered more complicated and fragile than required.
Adding an entry for `hieradata` into the `Puppetfile` is more reliable, easier to implement, and
allows the `Puppetfile` be the canonical description of the environment.

### `moduledir` method
In R10k 1.4.0, it became possible to use the `moduledir` directive to direct R10k module entries
to install modules in a non-standard relative path. The `moduledir` directive can be placed before
before all `mod` entries in a `Puppetfile` to redirect the installation location. Though it is not directly
documented, the `moduledir` directive can be used to "reset" the path `mod` entries install into at any point
in a `Puppetfile`. In the past this has been used to achieve a similar effect as the `install_path` argument
on the `mod` entry as detailed in this standard.

```ruby
mod 'trlinkin/nsswitch'

moduledir 'hieradata'

mod 'site_data',
  :git => 'https://git.company.net/team/site_data.git'
```

This method is discouraged as it is an undocumented, unsupported use of `moduledir`. This method is overall
fragile and could no longer be available at any time in the future without warning. When encountered, it should
be replaced with one of the acceptable methods detailed in this standard.

## Feedback / Ideas for Improvement

* Loosely following RFC2119 for wording of things like 'MUST', 'SHALL', and the like: https://www.ietf.org/rfc/rfc2119.txt
* Feedback can be provided as an issue on this [Git repository](https://github.com/puppetlabs/best-practices/issues).


# Other Information

* https://support.puppet.com/hc/en-us/articles/226118987
* https://docs.puppet.com/pe/2016.5/cmgmt_puppetfile.html#specify-install-paths-for-repositories
* https://docs.puppet.com/pe/latest/code_mgr.html
