# ⚠ WARNING ⚠

This repository is no longer mantained since ~ 2019. While some of these documentation might still be useful, much of it is obsolete or out of date.

For a maintained collection of similar information, please see [Patterns and Tactics](https://www.puppet.com/docs/patterns-and-tactics/latest/patterns-and-tactics.html).
# Scaling Puppet Enterprise

## Summary

This standard defines how to determine bottlenecks in PE and think about adding compile masters or adding resources to an existing PE installation.

## Expectations

This best practice relies on the [Puppet Enterprise metrics collection best practice](https://github.com/puppetlabs/best-practices/blob/master/puppet-enterprise-metrics-collection.md) and assumes the user will have metrics that can be trended over time.  

There is also an expectation that the user will have already performed tuning to have the correct amount of heap, number of JRubies, command processing threads, etc… Tuning should be completed before scaling with additional hardware.  For simplicity tuning and scaling are treated as separate topics, we assume you already tuned appropriately then metrics can indicate the need for more hardware resources.

## Standard Details

### Preferred Option

There are three main bottlenecks in PE:

 - Catalog compilation
 - PostgreSQL ( the backend database to PuppetDB )
 - PuppetDB service

You should assume that you will run into a bottleneck in catalog compilation before you will run into the need to scale PostgreSQL.  After you have added compile masters to address the need for more catalog compilation then you may need to add resources to the node serving as your PostgreSQL instance.  Finally, after a sufficient amount of load has been reached you may reach a bottleneck in the PuppetDB service’s ability to process commands and place them in PostgreSQL.

#### Determining the need to add compile masters

Avg free JRubies is the primary metric to use when determining when you need to add compile masters.  When you deploy new nodes, the initial pluginsync they need to perform will cause an increased need for JRubies that should then reduce after they have received the initial pluginsync.  Load avg and avg free JRubies should be tied at the hip except for when you are doing that initial pluginsync in which case avg free JRubies will be near 0 and you will not be maxing out the load. After steady state is achieved and most JRubies are compiling catalogs then avg free JRubies and load avg will be tightly correlated again.  Based on the number of compile masters you have you can reasonably estimate the number of nodes a compile master must accommodate.  Assume you have 10K nodes and 3 compile masters with 1 avg free JRuby on each of the compile masters.  

10K / 3 = 3,333 nodes per compile master 

The magic number for how many nodes a compile master can support changes over time as puppet code is added, catalogs change, and/or facts are added.  

When you are running compile masters with more than 1 avg free JRuby then you have capacity to add more nodes to your PE installation.  In other words, you should add additional compile masters when you have less than 1 avg free Jruby.  

Another useful calculation is nodes per JRuby.  Assume 10k nodes with 3 compile masters with 8 JRubies each and averaging 1 free JRuby on each compile master.  

10k / 24 = 416 nodes per JRuby. 

If you’d like to add 2.5K nodes to your installation you will need 2.5k / 416 = 6 JRubies which can be accommodated with 1 additional compile master with 8 JRubies.  

#### Determining the need to add resources to your MoM 

In the above compile master section we naively assume that you can infinitely horizontally scale compile masters but there are central bottlenecks.  The MoM has the Console, PuppetDB and PostgreSQL located on it and along the way you will need to add CPU and RAM to this machine.  The easiest thing to track is the load avg / CPU use on the MoM and as CPU use gets closer to 100% or load avg approaches the number of available cores then you will want to add resources to keep up with demand.  

There are various metrics in PuppetDB that will show a slowdown as resource capacity is reached on the MoM.  Commands / sec, replace facts, replace catalog, and store report will all slow down as resource capacity is reached on the MoM.  If you see a trend of these getting longer and longer over time then two things should be considered.  VACUUM FULL or pg_repack of the tables in the pe-puppetdb database which optimizes performance of queries on those tables.  If that does not yield significant enough improvements in storage time then it will be time to increase the available CPU and RAM to the MoM.

#### Determining a need for additional PuppetDB service nodes

A rule of thumb for the PuppetDB service is that at most a single PuppetDB service can handle 50 commands / sec which is roughly 30K nodes on a 30 minute runinterval.  The author reached this upper limit in PE 2015.3 so it may have increased but for practical purposes the size of a machine needed to handle this load is already reaching the limits of practicality for customers.  

### Alternate Options

In order to mediate a known spike in JRuby use for pluginsync, you can use bulk_pluginsync which reduces the amount of work that pluginsync does by having your agent installation script download a tarball of the plugins from your master and extract it into place before installing the agent. Bulk_pluginsync can be used in the following ways: 

1. PE2019.0 - enabled by default
2. PE2018.1.5+ - Can be enabled https://puppet.com/docs/pe/2018.1/pe_enhancements.html#nix-bulk-plugin-sync-with-the-install-script-2018-1-5
3. Prior to PE 2018.1.5, install the [bulk_pluginsync module](https://forge.puppet.com/npwalker/bulk_pluginsync) 

### Discouraged Options


## Feedback / Ideas for Improvement
