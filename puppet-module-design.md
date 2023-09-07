# ⚠ WARNING ⚠

This repository is no longer mantained since ~ 2019. While some of these documentation might still be useful, much of it is obsolete or out of date.

For a maintained collection of similar information, please see [Patterns and Tactics](https://www.puppet.com/docs/patterns-and-tactics/latest/patterns-and-tactics.html).
# Puppet Module Design

## Summary

The standard provides the best practices for designing Puppet modules.

## Expectations

The standard applies to any Puppet developer writing Puppet code. This standard
is applicable to Puppet 5 and later.

## Best Practice Details

This best practices  provides a method of creating modules. This does not cover
how to write classes, defined types, functions or facts; but covers the best
practices for these module elements.

### Preferred Option

#### Module Design

The following are basic design best practices:

* All Puppet manifests (\*.pp files containing classes or defined types) must
  be under the `manifests` directory.
* Modules must only contain resources that are related to the issue the module
  is solving and not resources that do not immediately relate to the problem.
E.g. a `phpmyadmin` module would not contain resources to manage the
installation of `Apache` or `MySQL`, only resources to provide configuration for
these that relates to `phpmyadmin`.
* Each module must contain a `README.md` in valid markdown format, containing at
  the minimum a description of the module and how to use the module.
* Each class and defined type must include documentation at the top of the file
  that describes the purpose of the class or defined type, as a minimum, in a
format that Puppet Strings can interpret.
* Each module must contain a `metadata.json` file containing information,
  version, dependancies and intended operating systems for the module.
* Public-facing classes or defined types must have their parameters documented
  at the top of the file, under the description, in a format that Puppet Strings
can interpret.
* Modules should be named after the technology it is managing, such as `apache`
  or `iis` and not a name that includes a verb, such as `manage_apache` or
`enable_iis`.
* Modules should contain examples of how the module is intended to be used in
  different scenarios.
* Puppet code should adhere to the Puppet Style Guide

#### Module Layout

With the release of Puppet 5 the layout of a Puppet module has changed.

The recommended module layout is as follows:

```shell
<MODULE NAME>
├── data/
├── examples/
├── facts.d/
├── files/
├── functions/
├── hiera.yaml
├── lib/
├── manifests/
├── metadata.yaml
├── README.md
├── spec/
├── tasks/
├── templates/
└── types/
```

The `data` directory contains the in-module Hiera data.

The `examples` directory contains examples of how to declare and use the
module's classes and defined types.

The `facts.d` directory contains external facts that are synced to all nodes.

The `files` directory contains static files that managed nodes can download.
This directory may contain subdirectories.

The `functions` directory contains functions written in the Puppet language.

The `hiera.yaml` contains the configuration of Hiera within the module.

The `lib` directory contains plug-ins, such as custom facts, functions, types
and providers. The contents of this directory are synced to all nodes.

The `manifests` directory must contain all the Puppet manifests and can contain
further directories describing implementations, such as `client` or `server`.

The `metadata.yaml` file contains metadata relating to the module.

The `README.md` file contains documentation for the module, including
description, setup, usage, references and limitations.

The `spec` directory contains tests for Puppet code in the `manifests` directory
and plug-ins within the `lib` directory.

The `tasks` directory contains the code and metadata for Puppet Tasks.

The `templates` directory contains templates that the module's manifests can
use.

The `types` directory contains type aliases.

Not all of the above directories will be required when creating a module. The
Puppet Development Kit (PDK) should be used to create new modules, classes and
tasks as this tool creates the correct directory structure. The PDK help
provides information on using this tool to create new modules, classes and
tasks.

#### Basic Modules

A basic module is a module where there is a single class that contains all the
resources for that module.

When creating basic modules the entry point should be the class named after the
module (i.e. the `init.pp` file). This file can contain all the resources
required to implement the solution that the modules addresses.

The main class of a module is its interface point and should be the only
parameterised class if possible, this allows the module author to control the
usage of the entire module with the inclusion of a single class.

#### More Advanced Modules

In many cases the modules will be more complex due to the issue they are
addressing.

There is often a need to separate installation, configuration and service
elements of a module into their own classes, as follows:

```shell
<MODULE NAME>
└── manifests/
    ├── config.pp
    ├── init.pp
    ├── install.pp
    └── service.pp
```

In some cases the module will have different implementations, such as `server`
and `client` elements. If this is the case the manifests for each implementation
should reside under a directory describing that implementation within the
`manifests` directory of the module. The class defining the implementation
should still reside under the root of the manifests directory and is the
interface point for the implementation. The following is an example of the
implementation:

```shell
<MODULE NAME>
└── manifests/
    ├── client/
    │   ├── config.pp
    │   └── install.pp
    ├── client.pp
    ├── server/
    │   ├── config.pp
    │   ├── install.pp
    │   └── service.pp
    └── server.pp
```

In this case there is often no requirement for an `init.pp` file as the entry
points are the `client.pp` and `server.pp` files.

#### Class Parameters and In-Module Data

Parameters are the API interface for the module classes. Each parameter must be
documented in any public-facing class in a format that Puppet Strings can
interpret, as can be seen below:

```Puppet
# ntp
#
# Main class, includes all other classes.
#
# @param authprov
#   Enables compatibility with W32Time in some versions of NTPd (such as Novell DSfW). Default value: undef.
#
# @param broadcastclient
#   Enables reception of broadcast server messages to any local interface. Default value: false.
#
# @param config
#   Specifies a file for NTP's configuration info. Default value: '/etc/ntp.conf' (or '/etc/inet/ntp.conf' on Solaris).
```

When writing classes that have parameters the parameters should include the
variable type where possible, such as:

```puppet
# ntp/manifests/init.pp
class ntp (
  Array[String]           $servers       = ['0.pool.ntp.org', '1.pool.ntp.org', '2.pool.ntp.org', '3.pool.ntp.org'],
  Boolean                 $iburst_enable = false,
  Optional[Array[String]] $fudge         = [ ],
  ...
) {

  ...

}
```

With the release of Hiera 5 the `params` design pattern is no longer required in
most situations and has been replaced with in-module data. This requires a
module `hiera.yaml`, which contains the hierarchy and backends for Hiera within
the module. The following is an example of the `hiera.yaml` file:

```yaml
# ntp/hiera.yaml
---
version: 5

defaults:
  datadir: 'data'
  data_hash: 'yaml_data'

hierarchy:
  - name: 'Full Version'
    path: '%{facts.os.name}-%{facts.os.release.full}.yaml'

  - name: 'Major Version'
    path: '%{facts.os.name}-%{facts.os.release.major}.yaml'

  - name: 'Distribution Name'
    path: '%{facts.os.name}.yaml'

  - name: 'Operating System Family'
    path: '%{facts.os.family}.yaml'

  - name: 'common'
    path: 'common.yaml'
```

The hierarchy should only address differences in operating systems and not data
specific to an environment or installation. Including a hierarchy using only
operating system and not site specific information allows the module to be
reusable. Information relating to sites, environments or installations should be
provided at the `Profile` layer.

The lowest-priority default value should be defined as a parameter default
in-class, rather than in data-in-modules common.yaml. This is to ensure that
there is clarity when reading the code or `puppet strings` generated
documentation about which parameters are required, vs. which parameters have
default values provided by the module. It is an unfortunate shortcoming that
nothing will clearly indicate when default values shown may be overridden by
more specific ones from data-in-modules.

#### Puppet Tasks

Puppet Tasks can be written in any language that is supported on the node. All
Task files must reside within the `tasks` directory. Each Task must have an
associated metadata file (`<TASK NAME.json>`) that contains the description,
inputs and outputs of the Task.

#### Module Testing

All classes, defined types, and custom extensions should have associated
testing. Unit testing should be included as a minimum and the tests should
address common scenarios the module addresses, including operating systems,
parameter variations and error handling.

### Alternative Options

In some cases the in-module Hiera 5 pattern is not suitable, in these cases the
`params` design pattern should be used. The class containing the parameters for
the module must be called `params` and must only contain variables and no
resources. The `params` class shall not have any parameters. Any conditional
logic within the `params` class must have a `default` clause with a `fail`
statement to issues with unsatisfied test cases. The `params` class contains
variables that can be referenced by classes in the module. If a class is to use
the variables from the `params` class as defaults for the classes' parameters
the class must inherit the `params` class:

```puppet
# params.pp
class apache::params {

  case $facts['os']['family'] {
    'RedHat': {
      $apache_package = 'httpd'
      ...
    }
    'debian': {
      $apache_package = 'apache2'
      ...
    }
    default: {
      fail("The operatng system ${facts['os']['family']} is not supported by this module")
    }
  }
}

# init.pp
class apache (
  String $apache_package = $apache::params::apache_package,
  ...
) inherits apache::params {

  ...

}
```

## Feedback / Ideas for Improvement

* Loosely following RFC2119 for wording of things like 'MUST', 'SHALL', and the
  like: https://www.ietf.org/rfc/rfc2119.txt
* Feedback can be provided as an issue on this [Git
  repository](https://github.com/puppetlabs/best-practices/issues).

## Other Information

* https://puppet.com/docs/puppet/latest/modules_fundamentals.html
* https://puppet.com/docs/puppet/latest/bgtm.html
* https://puppet.com/docs/puppet/latest/plugins_in_modules.html
* https://puppet.com/docs/puppet/latest/modules_metadata.html
* https://puppet.com/docs/puppet/latest/modules_documentation.html
* https://puppet.com/docs/puppet/latest/functions_basics.html
* https://puppet.com/docs/puppet/latest/hiera_intro.html
* https://puppet.com/docs/puppet/latest/style_guide.html
