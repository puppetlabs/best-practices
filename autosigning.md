# Autosigning

## Summary

This standard defines the recommended method for automatically signing Puppet certificates. As in the Puppet documentation, this standard advises against [naïve](https://puppet.com/docs/puppet/latest/ssl_autosign.html#enabling-nave-autosigning) and [basic](https://puppet.com/docs/puppet/latest/ssl_autosign.html#basic-autosigning-autosignconf) autosigning, and recommends the use of [policy-based autosigning](https://puppet.com/docs/puppet/latest/ssl_autosign.html#policy-based-autosigning) using the [danieldreier/autosign](https://forge.puppet.com/danieldreier/autosign) module. This allows for the generation of single-use, limited-scope autosigning tokens that can be integrated with Puppet's RBAC service.


## Expectations

Customers expressing interest in autosigning, who wish to allow users outside the main Puppet team, or automated processes, to automatically sign the certificates of newly provisioned machines should be directed to use this standard.

The standard may be too complex for small customers who build few new servers and for which manually signing certificates is not an issue. However it will be relevant for customers that wish to sign certificates automatically, while still retaining Role Based Access Control and Active Directory/LDAP integration.

## Standard Details

### Preferred Option

#### Use the danieldreier/autosign autosign module to manage the generation and validation of signing tokens

The [danieldreier/autosign](https://forge.puppet.com/danieldreier/autosign) module (version `>= 0.2.0`) is recommended for managing automatic certificate signing. The module has the following features:

  - Tokens can be generated in the GUI using a task
  - Tokens can be generated using a REST API using the Orchestration API to run the task
  - Access to token generation can be controlled via Puppet's RBAC and therefore LDAP integrated by controlling access to the task
  - Tokens can be generated in code using a puppet function which can be used when Puppet is populating AWS `userdata` or similar
  - Validation also supports static passphrases and custom validation backends
  - Autosign executables are installed and configured automatically

Control around who is and is not allowed to generate autosigning tokens can be controlled by giving `autosign::generate_token` task execution permissions to specific groups.

**Generating Tokens:**

Tokens should be generated using the `autosign::generate_token` task. This task requires that a certname or regular expression be provided, and be to run against the Puppet Server. The output of the task will be [JSON Web Token](https://jwt.io/) which the `autosign` gem on the Puppet Server will recognise. The token will only be valid if it is used against a CSR with the name provided. The [Puppet Orchestration API](https://puppet.com/docs/pe/2017.3/orchestrator/orchestrator_api_commands_endpoint.html#post-command-task) can also be used to obtain tokens if this process needs to be automated or scripted.

Once tokens have been generated using the task, they should be inserted as the `challengePassword` attribute and will therefore be signed using the the autosign validator.

### Alternate Options

**Create a custom autosign validator.**

If customers have requirements that are not addressed by the Preferred Option, it is recommended that they consider writing a custom autosign validator. Autosign tokens are simply [JSON Web Tokens](https://jwt.io/) meaning that they can be embedded with any arbitrary data, as long as they use the HS256 algorithm and are signed with the correct secret. Custom autosign validators could validate any arbitrary data that is embedded within the token without the need for writing too much custom code, and would not break the existing functionality.

**Use a multiplexer for compiler autosigning**

As a workaround to [SERVER-1005](https://tickets.puppetlabs.com/browse/SERVER-1005), a multiplexer configuration may be used to  autosign certificates with dns-alt-name or authorization extensions. (for the workaround, a multiplexer script must shell out to `puppet cert sign --allow-dns-alt-names --allow-authorization-extensions` to sign the certificate out-of-band).

There is not currently a standard multiplexer script for this purpose.

The multiplexer script should return false for normal agent certificate requests.

This alternative should only be implemented when automation is needed for spinning up compilers.

### Discouraged Options

**Rolling something yourself.**

Completely writing an autosign system from scratch is discouraged as it is a lot of work given that there are already well-tested, extensible solution being used in production at many customers. If however an autosign system was written from scratch it should have the following attributes:

  - Tokens should be single use by default
  - Tokens should be able to expire
  - Tokens should have a limited scope i.e. only valid for one certname
  - Token generation should be limited to a subset of users, ideally defined within AD

## Feedback / Ideas for Improvement

Most of the functionality mentioned above was implemented in version `0.2.0` of the [danieldreier/autosign](https://forge.puppet.com/danieldreier/autosign) module. Any changes in technical implementation should be submitted as a pull request to that repository. Other feedback can be sent do dylan.ratcliffe@puppet.com or provided as a pull request to this repository.

## Other Information

N/A
