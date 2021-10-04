---
layout: post
title: "Gitops in Kafka"
date: 2021-10-03
---

It is not easy to manage Infrastructure as going wrong in any of the steps worsen the situation. Also, replicating the same set of configurations for an additional cluster is quite tedious. Here comes the Infrastructure as Code(IaC) handy. It helps us to provision the same Infrastructure across environments with the help of a source code. The IaC generates a binary of the source code. Then the IaC provisions the same infrastructure with the binary across environments every time it is applied and they are also Idempotency compliant. 

A few of the IaC tools are Terraform, Ansible, Chef etc.

<!--more-->

Kafka is a framework to implement Stream Processing. The data is written by many processes called produces and the same are read by consumers. The data are partitioned into different partitions called topics. Kafka runs on a cluster of one or more servers called brokers and the partitions are distributed across the cluster.

The infrastructure of Kafka can be provisioned with many tools. Here we will see julie-ops. 

[julie-ops][julie-ops] tool helps us to provision Kafka related tasks in Confluent Cloud Infrastructure as a code. 
The related tasks are usually [Topics][Topics], [Access Control][Access Control], [Handling schemas][Handling schemas],
[ksql artifacts][ksql artifacts] etc. 
All these tasks are configured as [topologies][topologies] in julie-ops.

## Pre-Requisites

- You need julie-ops installed locally or in docker
- Topologies
- Write the following configurations to a `.properties` file to connect to Kafka cluster:
  ```
    bootstrap.servers=\"<BOOTSTRAP_SERVER_URL>\" \n
    security.protocol=SASL_SSL \n
    sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule   required username=\"<SASL_USERNAME>\"   password=\"<SASL_PASSWORD>\"; \n
    ssl.endpoint.identification.algorithm=https \n
    sasl.mechanism=PLAIN \n
    # Required for correctness in Apache Kafka clients prior to 2.6 \n
    client.dns.lookup=use_all_dns_ips \n
    # Confluent Cloud Schema Registry \n
    schema.registry.url=\"<SCHEMA_REGISTRY_URL>\" \n
    basic.auth.credentials.source=USER_INFO \n
    schema.registry.basic.auth.user.info=\"<SCHEMA_REGISTRY_API_KEY>\":\"<SCHEMA_REGISTRY_API_SECRET>\"
  ```

## How to run

```
julie-ops --broker <BROKERS> --clientConfig <PROPERTIES_FILE> --topology <TOPOLOGY_FILE>
```

Once the run is completed without any errors a successful run will look like

```
log4j:WARN No appenders could be found for logger (org.apache.kafka.clients.admin.AdminClientConfig).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
List of Topics:
<topics that are created>
List of ACLs:
<acls that are created>
List of Principles:
List of Connectors:
List of KSQL Artifacts:
Kafka Topology updated
```
[julie-ops]: https://julieops.readthedocs.io/en/latest/#
[Topics]: https://julieops.readthedocs.io/en/latest/futures/what-topic-management.html
[Handling schemas]: https://julieops.readthedocs.io/en/latest/futures/what-schema-management.html
[Access Control]: https://julieops.readthedocs.io/en/latest/futures/what-acl-management.html
[ksql artifacts]: https://julieops.readthedocs.io/en/latest/futures/what-ksql-management.html
[topologies]: https://julieops.readthedocs.io/en/latest/the-descriptor-files.html?highlight=topology
