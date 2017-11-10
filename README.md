# Docker ELK stack

[![Join the chat at https://gitter.im/deviantony/docker-elk](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/deviantony/docker-elk?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![Elastic Stack version](https://img.shields.io/badge/ELK-5.6.3-blue.svg?style=flat)](https://github.com/deviantony/docker-elk/issues/182)
[![Build Status](https://api.travis-ci.org/deviantony/docker-elk.svg?branch=master)](https://travis-ci.org/deviantony/docker-elk)

Run the latest version of the ELK (Elasticsearch, Logstash, Kibana) stack with Docker and Docker Compose.

It will give you the ability to analyze any data set by using the searching/aggregation capabilities of Elasticsearch
and the visualization power of Kibana.

Based on the official Docker images:

* [elasticsearch](https://github.com/elastic/elasticsearch-docker)
* [logstash](https://github.com/elastic/logstash-docker)
* [kibana](https://github.com/elastic/kibana-docker)

**Note**: Other branches in this project are available:

* ELK 5 with X-Pack support: https://github.com/deviantony/docker-elk/tree/x-pack
* ELK 5 in Vagrant: https://github.com/deviantony/docker-elk/tree/vagrant
* ELK 5 with Search Guard: https://github.com/deviantony/docker-elk/tree/searchguard

## Contents

1. [Requirements](#requirements)
   * [Host setup](#host-setup)
   * [SELinux](#selinux)
2. [Getting started](#getting-started)
   * [Bringing up the stack](#bringing-up-the-stack)
   * [Initial setup](#initial-setup)
3. [Configuration](#configuration)
   * [How can I tune the Kibana configuration?](#how-can-i-tune-the-kibana-configuration)
   * [How can I tune the Logstash configuration?](#how-can-i-tune-the-logstash-configuration)
   * [How can I tune the Elasticsearch configuration?](#how-can-i-tune-the-elasticsearch-configuration)
   * [How can I scale out the Elasticsearch cluster?](#how-can-i-scale-up-the-elasticsearch-cluster)
4. [Storage](#storage)
   * [How can I persist Elasticsearch data?](#how-can-i-persist-elasticsearch-data)
5. [Extensibility](#extensibility)
   * [How can I add plugins?](#how-can-i-add-plugins)
   * [How can I enable the provided extensions?](#how-can-i-enable-the-provided-extensions)
6. [JVM tuning](#jvm-tuning)
   * [How can I specify the amount of memory used by a service?](#how-can-i-specify-the-amount-of-memory-used-by-a-service)
   * [How can I enable a remote JMX connection to a service?](#how-can-i-enable-a-remote-jmx-connection-to-a-service)

## Requirements

### Host setup

1. Install [Docker](https://www.docker.com/community-edition#/download) version **1.10.0+**
2. Install [Docker Compose](https://docs.docker.com/compose/install/) version **1.6.0+**
3. Clone this repository

### SELinux

On distributions which have SELinux enabled out-of-the-box you will need to either re-context the files or set SELinux
into Permissive mode in order for docker-elk to start properly. For example on Redhat and CentOS, the following will
apply the proper context:

```bash
$ chcon -R system_u:object_r:admin_home_t:s0 docker-elk/
```

## Usage

### Bringing up the stack

Start the ELK stack using `docker-compose`:

```bash
$ docker-compose up
```

You can also choose to run it in background (detached mode):

```bash
$ docker-compose up -d
```

Give Kibana about 2 minutes to initialize, then access the Kibana web UI by hitting
[http://localhost:5601](http://localhost:5601) with a web browser.

By default, the stack exposes the following ports:
* 5000: Logstash TCP input.
* 9200: Elasticsearch HTTP
* 9300: Elasticsearch TCP transport
* 5601: Kibana

**WARNING**: If you're using `boot2docker`, you must access it via the `boot2docker` IP address instead of `localhost`.

**WARNING**: If you're using *Docker Toolbox*, you must access it via the `docker-machine` IP address instead of
`localhost`.

Now that the stack is running, you will want to inject some log entries. The shipped Logstash configuration allows you
to send content via TCP:

```bash
$ nc localhost 5000 < /path/to/logfile.log
```

## Initial setup

### Default Kibana index pattern creation

When Kibana launches for the first time, it is not configured with any index pattern.

#### Via the Kibana web UI

**NOTE**: You need to inject data into Logstash before being able to configure a Logstash index pattern via the Kibana web
UI. Then all you have to do is hit the *Create* button.

Refer to [Connect Kibana with
Elasticsearch](https://www.elastic.co/guide/en/kibana/current/connect-to-elasticsearch.html) for detailed instructions
about the index pattern configuration.

#### On the command line

Run this command to create a Logstash index pattern:

```bash
$ curl -XPUT -D- 'http://localhost:9200/.kibana/index-pattern/logstash-*' \
    -H 'Content-Type: application/json' \
    -d '{"title" : "logstash-*", "timeFieldName": "@timestamp", "notExpandable": true}'
```

This command will mark the Logstash index pattern as the default index pattern:

```bash
$ curl -XPUT -D- 'http://localhost:9200/.kibana/config/5.6.2' \
    -H 'Content-Type: application/json' \
    -d '{"defaultIndex": "logstash-*"}'
```

## Configuration

**NOTE**: Configuration is not dynamically reloaded, you will need to restart the stack after any change in the
configuration of a component.

### How can I tune the Kibana configuration?

The Kibana default configuration is stored in `kibana/config/kibana.yml`.

It is also possible to map the entire `config` directory instead of a single file.

### How can I tune the Logstash configuration?

The Logstash configuration is stored in `logstash/config/logstash.yml`.

It is also possible to map the entire `config` directory instead of a single file, however you must be aware that
Logstash will be expecting a
[`log4j2.properties`](https://github.com/elastic/logstash-docker/tree/master/build/logstash/config) file for its own
logging.

### How can I tune the Elasticsearch configuration?

The Elasticsearch configuration is stored in `elasticsearch/config/elasticsearch.yml`.

You can also specify the options you want to override directly via environment variables:

```yml
elasticsearch:

  environment:
    network.host: "_non_loopback_"
    cluster.name: "my-cluster"
```

### How can I scale out the Elasticsearch cluster?

Follow the instructions from the Wiki: [Scaling out
Elasticsearch](https://github.com/deviantony/docker-elk/wiki/Elasticsearch-cluster)

## Storage

### How can I persist Elasticsearch data?

The data stored in Elasticsearch will be persisted after container reboot but not after container removal.

In order to persist Elasticsearch data even after removing the Elasticsearch container, you'll have to mount a volume on
your Docker host. Update the `elasticsearch` service declaration to:

```yml
elasticsearch:

  volumes:
    - /path/to/storage:/usr/share/elasticsearch/data
```

This will store Elasticsearch data inside `/path/to/storage`.

**NOTE:** beware of these OS-specific considerations:
* **Linux:** the [unprivileged `elasticsearch` user][esuser] is used within the Elasticsearch image, therefore the
  mounted data directory must be owned by the uid `1000`.
* **macOS:** the default Docker for Mac configuration allows mounting files from `/Users/`, `/Volumes/`, `/private/`,
  and `/tmp` exclusively. Follow the instructions from the [documentation][macmounts] to add more locations.

[esuser]: https://github.com/elastic/elasticsearch-docker/blob/016bcc9db1dd97ecd0ff60c1290e7fa9142f8ddd/templates/Dockerfile.j2#L22
[macmounts]: https://docs.docker.com/docker-for-mac/osxfs/

## Extensibility

### How can I add plugins?

To add plugins to any ELK component you have to:

1. Add a `RUN` statement to the corresponding `Dockerfile` (eg. `RUN logstash-plugin install logstash-filter-json`)
2. Add the associated plugin code configuration to the service configuration (eg. Logstash input/output)
3. Rebuild the images using the `docker-compose build` command

### How can I enable the provided extensions?

A few extensions are available inside the [`extensions`](extensions) directory. These extensions provide features which
are not part of the standard Elastic stack, but can be used to enrich it with extra integrations.

The documentation for these extensions is provided inside each individual subdirectory, on a per-extension basis. Some
of them require manual changes to the default ELK configuration.

## JVM tuning

### How can I specify the amount of memory used by a service?

By default, both Elasticsearch and Logstash start with [1/4 of the total host
memory](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/parallel.html#default_heap_size) allocated to
the JVM Heap Size.

The startup scripts for Elasticsearch and Logstash can append extra JVM options from the value of an environment
variable, allowing the user to adjust the amount of memory that can be used by each component:

| Service       | Environment variable |
|---------------|----------------------|
| Elasticsearch | ES_JAVA_OPTS         |
| Logstash      | LS_JAVA_OPTS         |

To accomodate environments where memory is scarce (Docker for Mac has only 2 GB available by default), the Heap Size
allocation is capped by default to 256MB per service in the `docker-compose.yml` file. If you want to override the
default JVM configuration, edit the matching environment variable(s) in the `docker-compose.yml` file.

For example, to increase the maximum JVM Heap Size for Logstash:

```yml
logstash:

  environment:
    LS_JAVA_OPTS: "-Xmx1g -Xms1g"
```

### How can I enable a remote JMX connection to a service?

As for the Java Heap memory (see above), you can specify JVM options to enable JMX and map the JMX port on the docker
host.

Update the `{ES,LS}_JAVA_OPTS` environment variable with the following content (I've mapped the JMX service on the port
18080, you can change that). Do not forget to update the `-Djava.rmi.server.hostname` option with the IP address of your
Docker host (replace **DOCKER_HOST_IP**):

```yml
logstash:

  environment:
    LS_JAVA_OPTS: "-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.port=18080 -Dcom.sun.management.jmxremote.rmi.port=18080 -Djava.rmi.server.hostname=DOCKER_HOST_IP -Dcom.sun.management.jmxremote.local.only=false"
```
# Scaling out Elasticsearch

This project supports a single-node Elasticsearch cluster by default. Following the instructions in this page, you will be able to scale out that cluster by adding extra nodes.

![ES multi replicas](https://www.elastic.co/guide/en/elasticsearch/guide/master/images/elas_4404.png)

*image source: [Elasticsearch: The Definitive Guide » Replica Shards](https://www.elastic.co/guide/en/elasticsearch/guide/master/replica-shards.html)*

## Prerequisites

### Increase `vm.max_map_count`

One must increase the `vm.max_map_count` kernel setting on all Docker hosts running Elasticsearch in order to pass the [bootstrap checks](https://www.elastic.co/guide/en/elasticsearch/reference/current/bootstrap-checks.html) triggered by the production mode.
To do this follow the recommended instructions from the Elastic documentation: [Install Elasticsearch with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#docker-cli-run-prod-mode)

### Enable Zen discovery

Use the [`zen`](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-zen.html) discovery plugin instead of `single-node`, the default in the included [Elasticsearch configuration file](https://github.com/deviantony/docker-elk/blob/master/elasticsearch/config/elasticsearch.yml#L16). This can be changed whether directly in the configuration file or via an environment variable:

Example:
```yml
# docker-compose.yml

  elasticsearch:
    environment:
      # use 'zen' discovery plugin
      discovery.type: zen
```

## Discovery

Although the multicast plugin was [removed in Elasticsearch 5.2](https://www.elastic.co/guide/en/elasticsearch/reference/5.2/breaking_50_plugins.html#_multicast_plugin_removed), is still possible to leverage the Docker internal DNS together with the [unicast Zen discovery](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html) mechanism.

For that, simply set the `discovery.zen.ping.unicast.hosts` Elasticsearch setting to the name of your Elasticsearch compose service, either via an environment variable or in the `elasticsearch.yml ` configuration file.

Example:
```yml
# docker-compose.yml

  elasticsearch:
    environment:
      # use internal Docker round-robin DNS for unicast discovery
      discovery.zen.ping.unicast.hosts: elasticsearch
```

## Port mapping

The default docker-compose file uses static host port mapping for the `elasticsearch` service. This prevents service outscaling because a single port can be mapped only once on the host. Instead, you have to either disable port mapping completely, or let Docker map container ports to random host ports in order to prevent clashes.

Example:
```yml
# docker-compose.yml

  elasticsearch:
    ports:
        # map to random host ports instead of static ports, eg. 32000:9200
      - "9200"
      - "9300"
```

## Scaling out

Once the ELK stack is fully started, scale out the `elasticsearch` service:

```
❯ docker-compose scale elasticsearch=3
Creating and starting elk_elasticsearch_2 ... done
Creating and starting elk_elasticsearch_3 ... done
```

Your extra nodes should automatically join the current master:
```
❯ docker logs elk_elasticsearch_2 
...
[2017-03-14T15:51:08,123][INFO ][o.e.c.s.ClusterService   ] [hDXuwLj]
  detected_master {IFnVp82}{IFnVp82CT--XfrPcrU-ndg}{xBfP-4jNTx2FXVLPlMzLLw}{172.18.0.2}{172.18.0.2:9300},
  added {{8pF4rmm}{8pF4rmmoR4uk3pNTQuQ9qQ}{k__zZfnPTQa3-GMFD7_oAw}{172.18.0.6}{172.18.0.6:9300},{IFnVp82}{IFnVp82CT--XfrPcrU-ndg}{xBfP-4jNTx2FXVLPlMzLLw}{172.18.0.2}{172.18.0.2:9300},},
  reason: zen-disco-receive(from master [master {IFnVp82}{IFnVp82CT--XfrPcrU-ndg}{xBfP-4jNTx2FXVLPlMzLLw}{172.18.0.2}{172.18.0.2:9300} committed version [39]])
```

If you're using the `x-pack` branch, all nodes will show up in Kibana's Monitoring app:

![ES cluster Kibana](https://cloud.githubusercontent.com/assets/3299086/23909321/b2b339e8-08d6-11e7-9670-595a17ee9eb1.png)
