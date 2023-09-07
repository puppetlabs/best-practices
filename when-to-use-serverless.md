# ⚠ WARNING ⚠

This repository is no longer mantained since ~ 2019. While some of these documentation might still be useful, much of it is obsolete or out of date.

For a maintained collection of similar information, please see [Patterns and Tactics](https://www.puppet.com/docs/patterns-and-tactics/latest/patterns-and-tactics.html).

# When To Use Serverless Puppet

## Summary

Some users dislike the idea of using a centralized Puppet server and would rather distribute their codebase and enforce it locally with `puppet apply`. We strongly discourage this practice. This best practice will define why we specifically discourage serverless Puppet as a solution.

## Expectations

Users who have encountered blog posts or other online guidance suggesting a serverless deployment may use this best practice to understand the suitability of running without a Puppet server in their environment.

Note that this guideline does not apply to the running of `puppet apply` during development or testing.

## Best Practice Details

### Preferred Option

**Use a supported Puppet architecture, including a Puppet server.**

### Problems with Serverless Puppet

A significant proportion of the value Puppet provides as a configuration management tool is centralized control of Puppet managed systems. In the absence of a Puppet server, however, users inevitably either lose the benefits of central control, or must create bespoke re-implementation of services the Puppet server provides. There are significant maintenance costs for architecting and supporting your own solution to problems already solved by an off-the-shelf solution.

Additionally, when a Puppet server is not used every agent must be given full access to the source manifests (not required when using a server) as well as all data inputs required to build its Puppet catalog. This either constrains what information can be provided to agents, requires integration of separate tooling to distribute secrets to agents, or results in loss of the “principle of least privilege” security benefit Puppet enjoys through limiting information given to each individual agent to only what it needs to know.

Finally, beyond Puppet configuration management through manifests, when not using a Puppet server all other Puppet-provided services are effectively unavailable. This includes but is not limited to orchestration, tasks, and reporting.

Serverless is not one of the approved Puppet architectures, which limits the amount of available information there is on how to implement it.

### Situations where users might use Puppet without a server

Desired state is used for provisioning, but not afterwards. (e.g. "Here's a node that we know is good. What you do with it from now on is your business; don't call us if it breaks.")

#### HPC/Supercomputing

Some high performance computing environments have traditionally gravitated towards a serverlessless model, for instance when:
* nodes are immutable, and so Puppet is just confirming that desired state is enforced (e.g. services are running, tempfs isn't full, etc.) and configuration is handeled in some other way (e.g. an exported, read-only root file system) 
* CPU resources are at a premium, and the cost of running **any** non-essential load is too high
* Managed system count is large enough that a non-trivial puppet configuration with compile servers might be desired

High performance compute environments might also reboot/reconfigure an entire cluster at once in preparation for the next batch job which could present a thundering herd problem. (Puppet 4.0/Puppet Server 2.0 and newer have several features that mitigate thundering herds; Puppet 5 onwards even more so.)

HPC Systems are typically associated with (and dependant on) one or more low-latency, high-performance file systems, allowing all nodes to easily access Puppet code from a single (read-only) location. This source of truth reduces code distribution overhead associated with other use cases (e.g. shared nothing). Use of version control remains strongly encouraged; editing a file on the shared file system may seem quick and easy, knowing that all nodes will see the change immediately, but this approach brings several hazards and risks. The same file system can also be a destination for puppet run output, logs etc. This may simplify troubleshooting versus other use cases.

Some HPC sites use 'puppet apply' as part of their node health checks, to ensure each node is:
* in its desired state, healthy and otherwise ready to process the next job
* running the latest configuration
* configured consistently with its peers participating in the job

While this approach can have several benefits, it can also add significant complexity as node configuration is (typically) only updated at the start of a new job. (The nodes with the longest running jobs have the oldest configuration, and the ones with the just-started jobs have the newest, and every other node is some version in between.) This may make it more difficult to know when a specific configuration change is rolled out across **all** nodes in the entire cluster.

Please note that most of these concerns can be addressed simply and easily with Puppet 4.4 and newer features such as [static catalogs](https://puppet.com/docs/pe/2021.2/static_catalogs.html) and [direct change](https://puppet.com/docs/pe/2019.0/direct_puppet_a_workflow_for_controlling_change.html)-based workflow. An approach based on these features can strike a balance between having no desired state enforcement while jobs are running (e.g. jobs failing because a limited resource is depleted slowly during the run, which Puppet could identify and fix) while not requiring a significant Puppet infrastructure to maintain, even in the case of a thundering herd.

## Other Information

https://puppet.com/docs/pe/2021.4/pe_architecture_overview.html

https://puppet.com/docs/puppet/7/architecture.html
