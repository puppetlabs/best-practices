# When To Use Serverless Puppet

## Summary

Some users dislike the idea of using a centralized Puppet Server and would rather distribute their codebase and enforce it locally with `puppet apply`. We strongly discourage this practice. This best practice will define why we specifically discourage Serverless Puppet as a solution.

## Expectations

Users who have encountered blog posts or other online guidance suggesting a Serverless deployment may use this best practice to understand the suitability of running without a Puppet Server in their environment.

Note that this guideline does not apply to the running of `puppet apply` during development or testing.

## Best Practice Details

### Preferred Option

**Use a supported Puppet architecture, including a Puppet Server.**

### Problems with Serverless Puppet

A significant proportion of the value Puppet provides as a configuration management tool is centralized control of Puppet managed systems. In the absence of a Puppet erver, however, users inevitably either lose the benefits of central control, or must create bespoke re-implementation of services the Puppet Server provides. There are significant maintenance costs for architecting and supporting your own solution to problems already solved by an off-the-shelf solution.

Additionally, when a Puppet Server is not used every agent must be given full access to the source manifests (not required when using a Puppet Server) as well as all data inputs required to build its Puppet catalog. This either constrains what information can be provided to agents, requires integration of separate tooling to distribute secrets to agents, or results in loss of the “principle of least privilege” security benefit Puppet enjoys through limiting information given to each individual agent to only what it needs to know.

Finally, beyond Puppet configuration management through manifests, when not using a Puppet Server all other Puppet-provided services are effectively unavailable. This includes but is not limited to orchestration, tasks, and reporting.

Serverless is not one of the approved Puppet architectures, which limits the amount of available information there is on how to implement it.

### Situations where users might use Puppet without a Puppet Server

First boot configuration management is desired and Puppet is uninstalled immediately after the first run. This is sometimes seen in high performance computing environments where nodes are immutable and CPU utilization of an agent is seen as a problem.

High performance compute environments might also reboot/reconfigure an entire cluster at once in preparation for the next batch job which could present a thundering herd problem.

## Other Information

https://puppet.com/docs/pe/2018.1/overview/pe_architecture_overview.html

https://puppet.com/docs/puppet/5.5/architecture.html
