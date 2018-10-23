# When To Use Masterless Puppet

## Summary

Some users dislike the idea of using a centralized Puppet master and would rather distribute their codebase and enforce it locally with `puppet apply`. We strongly discourage this practice. This best practice will define why we specifically discourage masterless Puppet as a solution.

## Expectations

Users who have encountered blog posts or other online guidance suggesting a masterless deployment may use this best practice to understand the suitability of running without a Puppet master in their environment.

Note that this guideline does not apply to the running of `puppet apply` during development or testing.

## Best Practice Details

### Preferred Option

**Use a supported Puppet architecture, including a Puppet master.**

### Problems with Masterless Puppet

A significant proportion of the value Puppet provides as a configuration management tool is centralized control of Puppet managed systems. In the absence of a Puppet master, however, users inevitably either lose the benefits of central control, or must create bespoke re-implementation of services the Puppet master provides. There are significant maintenance costs for architecting and supporting your own solution to problems already solved by an off-the-shelf solution.

Additionally, when a Puppet master is not used every agent must be given full access to the source manifests (not required when using a master) as well as all data inputs required to build its Puppet catalog. This either constrains what information can be provided to agents, requires integration of separate tooling to distribute secrets to agents, or results in loss of the “principle of least privilege” security benefit Puppet enjoys through limiting information given to each individual agent to only what it needs to know.

Finally, beyond Puppet configuration management through manifests, when not using a Puppet master all other Puppet-provided services are effectively unavailable. This includes but is not limited to orchestration, tasks, and reporting.

Masterless is not one of the approved Puppet architectures, which limits the amount of available information there is on how to implement it.

### Situations where users might use Puppet without a master

First boot configuration management is desired and Puppet is uninstalled immediately after the first run. This is sometimes seen in high performance computing environments where nodes are immutable and CPU utilization of an agent is seen as a problem.

#### HPC/Supercomputing

High performance compute environments might also reboot/reconfigure an entire cluster at once in preparation for the next batch job which could present a thundering herd problem.

HPC Systems are typically associated with (and dependant on) one or more low-latency, high-performance file systems, allowing all nodes to easily access Puppet code from a single (read-only) location. This source of truth reduces code distribution overhead associated with other use cases (e.g. shared nothing). Use of version control remains strongly encouraged; editing a file on the shared file system may seem quick and easy, knowing that all nodes will see the change immediately, but this approach brings several hazards and risks. The same file system can also be a destination for puppet run output, logs etc. This may simplify troubleshooting versus other use cases.

Some HPC sites use 'puppet apply' as part of their node health checks, to ensure each node is:
* in its desired state, healthy and otherwise ready to process the next job
* running the latest configuration
* configured consistently with its peers participating in the job

While this approach can have several benefits, it can also add significant complexity as node config is (typically) only updated at the start of a new job. (The nodes with the longest running jobs have the oldest configuration, and the ones with the just-started jobs have the newest, and every other node is some version in between.) This may make it more difficult to know when a specific configuration change is rolled out across **all** nodes in the entire cluster.

## Other Information

https://puppet.com/docs/pe/2018.1/overview/pe_architecture_overview.html

https://puppet.com/docs/puppet/5.5/architecture.html
