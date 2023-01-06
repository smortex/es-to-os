# ElasticSearch to OpenSeach

A set of tools to migrate from [ElasticSearch](https://www.elastic.co/) to [OpenSearch](https://opensearch.org/).

## Context

These tools where build within our context. It might not be suitable to you:

* Single node ElasticSearch 6.8 "cluster";
* Limited disk space;
* Mostly logs and metrics indices:
  * Daily indices for logs send by [syslog-ng](https://www.syslog-ng.com/) as `logs-*`;
  * Daily, monthly and yearly indices for metrics (collected every 5s and persisted at different intervals) send by [riemann-tools](https://github.com/riemann/riemann-tools) and sampled by [samplerr](https://github.com/ccin2p3/samplerr) as `.samplerr-*`.
* A few hours of downtime is acceptable;
* Missing metrics during the downtime is acceptable;
* Missing logs is not acceptable.

## Requirements

* Basically enough disk space to hold a copy of ElasticSearch data.
