# ElasticSearch to OpenSeach

A set of tools to migrate from [ElasticSearch](https://www.elastic.co/) to [OpenSearch](https://opensearch.org/).

## Context

These tools where build within our context. It might need some adjustments to fit your site configuration:

* Single node ElasticSearch 6.8 "cluster";
* Limited disk space;
* Mostly logs and metrics indices:
  * Daily indices for logs send by [syslog-ng](https://www.syslog-ng.com/) as `logs-*`;
  * Daily, monthly and yearly indices for metrics (collected every 5s and persisted at different intervals) send by [riemann-tools](https://github.com/riemann/riemann-tools) and sampled by [samplerr](https://github.com/ccin2p3/samplerr) as `.samplerr-*`.
* A few hours of downtime is acceptable;
* Missing metrics during the downtime is acceptable;
* Missing logs is not acceptable;
* Before the migration:
  * ElasticSearch is locally available as `http://127.0.0.1:9200`;
  * Kibana is locally available as `http://127.0.0.1:5601`;
* During the migration:
  * OpenSearch will be locally available as `https://127.0.0.1:9201`;
  * OpenSearch-Dashboards will be locally available as `http://127.0.0.1:5602`;
* After the migration:
  * OpenSearch will be locally available as `https://127.0.0.1:9200` (note that we switched to HTTPS);
  * OpenSearch-Dashboards will be locally available as `http://127.0.0.1:5601`;

## Requirements

* Basically enough disk space to hold a copy of ElasticSearch data.

## How-to

### Prerequisites

#### Install OpenSearch

* [ ] Install OpenSearch;
* [ ] Adjust `config/opensearch.yml`:
```yaml
http.port: 9201
reindex.remote.whitelist: "localhost:9200"
```
* [ ] Start OpenSearch

#### Install OpenSearch-Dashboards

* [ ] Install OpenSearch-Dashboards
* [ ] Adjust `config/opensearch_dashboards.yml`:
```yaml
server.port: 5602
opensearch.hosts: [https://localhost:9201]
```
* [ ] Start OpenSearch-Dashboards

#### Prepare Kibana

* [ ] Export Kibana objects: Management → Saved Objects → Export XXX objects;
* [ ] Check that all indices are open `GET _cat/indices?s=index&v`;
  * If indices are closed, open them with `POST logs-2022.01.23/_open`.

#### Setup OpenSearch-Dashboards

* [ ] Import Kibana objects into OpenSearch-Dashboards: Stack Management → Saved Objects → Import (it takes a while);
* [ ] Adjust the default index: Stack Management → Index Patters → logs-\* → Set as default index;
* [ ] In OpenSearch-Dashboards, Index Management → Index Policies → Create policy:
  * Name: `logs-rotation`;
  * ISM Templates: `logs-*`;
  * States:
    * `initial`. Transition: 1. Destination state expired; Transition trigger logic: Minimum index age is 365d;
    * `expired`. Actions: 1. Delete.

#### Migrate idle data

* [ ] Run `es-to-os`:
```sh-session
opensearch@localhost ~/es-to-os % ./es-to-os
[+] Setup index template logs...
[+] Setup index template samplerr...
[+] Reindexing logs-2022.01.23...                0:01:15
[+] Reindexing logs-2022.04.16...                0:02:29
[+] Reindexing logs-2022.02.18...                0:02:10
[+] Reindexing logs-2022.02.01...                0:02:22
[+] Reindexing logs-2022.07.05...                0:03:39
[+] Reindexing logs-2022.11.10...                0:02:53
[+] Reindexing logs-2022.11.08...                0:10:58
[+] Reindexing logs-2022.10.14...                0:10:14
[+] Reindexing logs-2022.07.06...                0:04:03
[+] Reindexing logs-2022.04.24...                0:02:29
[+] Reindexing logs-2022.10.11...                0:14:38
[+] Reindexing logs-2022.08.28...                0:02:33
[+] Reindexing logs-2022.12.05...                0:04:58
[+] Reindexing .samplerr-2023.01.01...           0:43:30
[+] Reindexing logs-2022.09.26...                0:03:50
[+] Reindexing logs-2022.05.22...                0:02:21
[+] Reindexing logs-2022.04.29...                0:03:41
[+] Reindexing logs-2022.07.17...                0:10:21
[+] Reindexing logs-2022.08.03...                0:03:48
[+] Reindexing logs-2022.02.27...                0:01:42
[+] Reindexing .samplerr-2021...
```

If the script is interrupted, or if an error is detected, some indexes may exist but only have a portion of the expected data.  Only indices that do not exist are reindexed, so these partial indices should be removed before trying to reindexing them. The `diff-es-os` script list indices that exist in both ElasticSearch and OpenSearch but have a different number of documents.

Note that we experienced situations where after reindex, OpenSearch did not report the expected number of documents.  Restarting OpenSearch fixed this issue.

#### Migrate live data

* [ ] Stop services that write data to ElasticSearch
* [ ] Run `es-to-os -f`:
```sh-session
opensearch@localhost ~/es-to-os % ./es-to-os -f
```
* [ ] Stop ElasticSearch
* [ ] Stop Kibana
* [ ] Stop OpenSearch
* [ ] Stop OpenSearch-Dashboards

#### Switch to OpenSearch


* [ ] Adjust `config/opensearch.yml`:
```yaml
http.port: 9200
#reindex.remote.whitelist: "localhost:9200"
```
* [ ] Start OpenSearch
* [ ] Adjust services that write data to OpenSearch to enable HTTPS;
* [ ] Start services that write data to OpenSearch

#### Switch to OpenSearch-Dashboards

* [ ] Adjust `config/opensearch_dashboards.yml`:
```yaml
server.port: 5601
opensearch.hosts: [https://localhost:9200]
```
* [ ] Start OpenSearch-Dashboards
