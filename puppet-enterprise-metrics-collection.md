# Puppet Enterprise Metrics Collection

## Summary

To ensure the information needed to troubleshoot performance issues is available when needed, it is recommended that Puppet Enterprise users set up a metrics gathering tool.  This standard defines how to gather, retain, and graph metrics for Puppet Enterprise components in a format that is easily shareable with Puppet Support.  

## Expectations

This is a standard that applies to all Puppet Enterprise customers.  

This standard assumes no existing performance monitoring infrastructure and the modules will configure influxdb and grafana for the user.  If there is desire to use existing tools for storing and graphing metrics then that will be left to the user to deduce from our examples.  

## Standard Details

### Preferred Option


#### Collecting metrics 

We recommend using the [puppet_metrics_collector](https://github.com/puppetlabs/puppetlabs-puppet_metrics_collector) module to collect metrics.  By default, the module stores metrics in `/opt/puppetlabs/puppet-metrics-collector`, and the data can be easily archived and shared with Support.

#### Storing and visualizing metrics 

For permanent visualization of metrics we recommend the [puppetlabs-puppet_metrics_dashboard](https://forge.puppet.com/puppetlabs/puppet_metrics_dashboard) module to store historical metrics in influxDB and visualize metrics with Grafana.  This option requires an additional server to run influxDB and Grafana.  After configuring influxDB with the module you will configure *puppet-metrics-collector* to directly export metrics into InfluxDB.

For temporary visualization of metrics we recommend [puppet-metrics-viewer](https://github.com/puppetlabs/puppet-metrics-viewer).  This option uses a docker container to allow visualizing the metrics on an ad-hoc basis but no data is permanently stored.

Both options use a set of example dashboards with the most useful metrics for troubleshooting scaling and performance issues.  

### Alternate Options

While the preferred option allows for long term analysis of the performance data, the developer dashboards can be used before long term solutions have been configured. The [PuppetDB dashboard](https://puppet.com/docs/puppetdb/5.2/maintain_and_tune.html#monitor-the-performance-dashboard) and the [Puppetserver developer dashboard](https://puppet.com/docs/puppetserver/5.2/puppet_server_metrics.html#accessing-the-developer-dashboard) provide a real time view of the current state of these services. This can be helpful in troubleshooting issues, but does not provide historical data. 

*Note*: The PuppetDB dashboard negatively impacts performance and should not be left open. Instead use it for troubleshooting specific incidents when you don't yet have longer term metrics gathering and visualization set up. 

### Discouraged Options

It is possible to collect metrics from JMX or direct export of graphite metrics from puppetserver.  These options are discouraged for their complexity and because the output is not readily shareable with Puppet Support or 3rd parties.  

## Feedback / Ideas for Improvement

Please submit issues to the projects below if you have ideas for improvement:

[puppet_metrics_collector](https://github.com/puppetlabs/puppetlabs-puppet_metrics_collector)  
[puppet_metrics_dashboard](https://github.com/puppetlabs/puppet_metrics_dashboard)  
[puppet-metrics-viewer](https://github.com/puppetlabs/puppet-metrics-viewer)  
