# Tuning Puppet Enterprise Services

## Summary

Puppet Enterprise is composed of multiple services, each with their own default settings, most of which are implemented as class parameters and some of which are only configured during installation. The default settings for these services are conservative, and not necessarily optimized for each infrastructure type and each potential combination of services on an infrastructure host.

## Expectations

Given the above, it's necessary to tune the settings of Puppet Enterprise services to maximize the use of available hardware resources after installation, and to reallocate resources after changing your infrastructure: for example: after adding compilers or replica Puppet Servers. It is also necessary to tune the settings in response to changes in the size and/or complexity of your infrastructure.

## Standard Details

Tuning Puppet Enterprise requires reviewing the documentation for each service, identifying the resources of each infrastructure host, and performing manual calculations for each infrastructure host and service.

The following documents tuning monolithic installations based on three standard sizes (4 CPU/8GB RAM, 8 CPU/16GB RAM, and 16 CPU/32GB RAM) of primary Puppet Server hardware, with variations when using compilers:

  * https://puppet.com/docs/pe/latest/configuring/tuning_monolithic.html

That public documentation was informed by the following internal documents:

  * [Document with some background](https://docs.google.com/document/d/1o3I5jSaonSj0-xCF7B9YNfUN_DhkeO-F-mBJeFnXPiw/edit)
  * [Spreadsheet with calculations](https://docs.google.com/spreadsheets/d/15FzysLcGkG8cEFFVuxE6nyca7x4YxQcn38wFWLXNVgk/edit)

> Note how the tuning monolithic guidelines differ from the defaults, and note how resources allocated for the Puppet Server and PuppetDB services on the primary Puppet Server are reallocated when using compilers.

The following documents tuning each service:

* https://puppet.com/docs/pe/latest/configuring/config_intro.html
* https://puppet.com/docs/pe/latest/configuring/config_java_args.html
* https://puppet.com/docs/pe/latest/configuring/config_puppetserver.html
* https://puppet.com/docs/pe/latest/configuring/config_console.html
* https://puppet.com/docs/pe/latest/configuring/config_puppetdb.html
* https://puppet.com/docs/pe/latest/installing/hardware_requirements.html
* https://puppet.com/docs/puppetserver/latest/tuning_guide.html

### Alternate Options

Support publishes tooling that outputs optimized settings for Puppet Enterprise services based upon infrastructure configuration and infrastructure host resources.

This tooling is available via the `puppet infrastructure tune` command in Puppet Enterprise 2018.1 and newer. It is also available for use with older versions of Puppet Enterprise as a standalone script via the following repository:

https://github.com/tkishel/pe_tune

This tooling reads `pe.conf` on the Primary Puppet Server, queries PuppetDB for node group membership to identify PE Infrastructure hosts, queries PuppetDB for processor and memory facts for each PE Infrastructure host, and outputs optimized settings for each service, in YAML format for use in Hiera. With a monolithic infrastructure, the output could be saved to a common/default YAML file. With a split infrastructure, the output would be saved to node-specific YAML files included in a node-specific hierarchy.

This tooling includes an option to output currently defined settings, in JSON format. Those settings may have been defined either in the Classifier (the Console) or in Hiera, with Classifier settings taking precedence over Hiera settings. Best practice is to define settings in Hiera (preferred) or the Classifier, but not both. The output of this option also identifies duplicate settings found in both the Classifier and Hiera as a diagnostic.

## Feedback / Ideas for Improvement

Please report issues with the standalone `pe_tune` script via its repository:

https://github.com/tkishel/pe_tune/issues

Please report issues with the `puppet infrastructure tune` command via the Puppet Enterprise project:

https://tickets.puppetlabs.com/projects/ENTERPRISE

Pull requests welcome!

## Other Information

Tuning Puppet Enterprise services can be considered one of a three-step process:

1. Tune services to maximize use of available resources.
1. Distribute agent load evenly across services and over time.
3. Determine if you need to add resources to increase capacity.

This document addresses tuning services.

The following addresses distributing agent load:

https://puppet.com/docs/pe/latest/installing_compilers.html#using-load-balancers-with-compilers
https://support.puppet.com/hc/en-us/articles/115004312247

The following addresses determining if you need to add resources:

https://github.com/puppetlabs/best-practices/blob/master/puppet-enterprise-metrics-collection.md
