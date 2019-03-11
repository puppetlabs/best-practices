## Summary

In Puppet, a control repository is a git repository that acts as the
configuration source of truth for all code and data for the organization
implementing it. This includes acting as the storage location for code, data,
and puppet modules, or as an index to the location of any of these if they are
owned by a third party (such as Forge modules).

This document identifies possible file contents of a control repository,
including explanations for why it may be helpful to include specific files in
the repository, or separate them into external repositories referenced by a
Puppetfile.

This document is not intedended to supercede the recommended default control
repository found [here](https://github.com/puppetlabs/control-repo).

This document assumes at least an intermediate level knowledge of the use of
Puppet, R10k, and the "Roles and Profiles" design pattern.

## Directory structure and contents

A typical control repository will look as follows:

```

├── data/
├── environment.conf
├── hiera.yaml
├── Puppetfile
├── README.md
├── manifests/
│   └── site.pp
└── site-modules/
    ├── profile/
    └── role/
```
### data

This directory contains the environment-layer-specific Hiera data. This is only
valid when using Hiera 5. This directory will generally contain the Hiera data
specific to an environment at a company. In some cases where for access control
purposes pieces or the entirety of this directory are included from other
sources via entries in the Puppetfile.

### environment.conf

This file is required at a minimum to set a modulepath that includes both `site-modules`
and `modules`. This cannot be set globally, and must be configured in this file
on a per-control-repo basis. Other settings may include the environment timeout
for the environment.

### hiera.yaml

This file is the configuration file of Hiera at the environment-layer and only
works in Hiera 5 format. Only envrionment-layer Hiera configuration exists in
this file. If there is not environment-layer Hiera required this file can be
omitted.

### Puppetfile

Though not strictly required, this file will most likely exist in all control
repositories. In the very rare case that no third party modules (either from a
public source, or from separate module repositories within the organization) are
included, this file may be omitted.

### README.md

The README.md file should include any important information on the modification,
ownership, and use of the control repository. What specific information is most
valuable here is up to the organization implementing it, but it should contain
at least the minimum to ensure that a user new to the organization but not
necessarily Puppet and R10k can get their bearings.

### manifests

This directory should in most cases contain only the site.pp file. This
directory may be omitted in a multi-tenant scenario where a central operations
team controls a global `site.pp` file for all control repository instances.
However, in nearly all cases, this directory will be included.

### manifests/site.pp

When using the recommended method of classification -- assigning nodes to single
roles in the Enterprise Console -- this file should contain a minimal amount of
Puppet code. This code should be limited to setting global resource defaults and
applying fact/hiera based classification. In rare cases, top scope values may be
declared here.

### site-modules

In most cases, the control repository will have a `site-modules` directory for modules
developed by the control repository owner. Modules in this directory are part of
the release cadence of the control repository itself. This directory may include
roles, profiles, and internally developed component modules as needed (see
below).

This directory is called "site-modules" by convention, and though no technical reason
exists that it must be named "site-modules," this name is preferred for consistency with
other literature.

### site-modules/role and site-modules/profile

In the majority of cases, these modules will be included in the control
repository. In some cases, it may be valid to include these as external modules
defined in the Puppetfile. See the relevant
[Roles](puppet-code-abstraction-roles.md) and
[Profiles](puppet-code-abstraction-profiles.md) best practices for more information.

### Component modules

Depending on the use case, the control repository will contain component modules
developed by the team which owns the control repository. Including internally
developed component modules in the control repository has several advantages and
disadvantages.

When a module is developed in-house and is not included directly in the control
repository, any changes during development necessitate at least two commits
before the module can be tested: first, a commit to the separate module
repository, and second, a commit to the control repository to update the
Puppetfile. This workflow can easily be seen as onerous to new users, and
therefore it is ideal at small shops to place new modules in the `site-modules`
directory.

If a module is developed by a team working independently from the team that owns
commits to the control repository, the module should be placed in its own
separate internal repository, and referenced by the Puppetfile. If a module will
be released to a third party lacking inner organizational context (such as the
Puppet Forge), the module should be kept separate from the control repository to
enable releases without copy/paste.

There is no formal standard or best practice insisting that component modules
are or are not included in the `site-modules` directory. The decision should be made
with the organization implementing Puppet code based on what is easiest and most
convenient for them. Further, it is perfectly acceptable to have a mix of
internally developed modules both in the `site-modules` directory and referenced in the
Puppetfile.

A common implementation pattern is to start module development directly in the
control repository, and move the module into its own repository if the module
becomes more complex or requires a separate development cadence.

### hieradata

The hieradata directory contains organization-specific data that is not relevant
outside of the context of a control repository. This directory will generally
contain the complete Hiera hierarchy used at a company, though cases may exist
where for access control purposes pieces or the entirety of this directory are
included from other sources.

The pattern of managing hieradata in a separate repository is both common and
acceptable, though should typically only be done in cases where there is an
identified need to separate access or the development cadence. This pattern is
described in the [hieradata repository](separate-hieradata-repository.md)
best practice.

Under Hiera 5, the data in this directory would be considered global and should
be deployed somehwere else.

### modules

The r10k tool will overwrite all contents of this directory with the modules
specified by the Puppetfile. If this directory does not exist, r10k or Code
Manager will automatically create it. Since git will not track an empty
directory, this will be the case.

### Other directories

The control repository may contain additional directories to support the
implementation and validation of Puppet code. This can include, but are not
limited to, general purpose scripts used in concert with the repository,
automated testing, and documentation.

## Feedback / Ideas for Improvement

* Loosely following RFC2119 for wording of things like 'MUST', 'SHALL', and the like: https://www.ietf.org/rfc/rfc2119.txt
* Feedback can be provided as an issue on this [Git repository](https://github.com/puppetlabs/best-practices/issues).

## Additional Reading

For foundational technical information, please see the documents linked below.

* https://puppet.com/docs/pe/latest/control_repo.html
* https://puppet.com/docs/puppet/latest/config_file_environment.html#modulepath
* https://puppet.com/docs/pe/latest/code_mgr_how_it_works.html
