---
layout: post
title: "Gitops in Kafka"
date: 2021-10-03
permalink: "/tech/gitops-in-kafka"
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

## Integrating with GitOps

GitOps is the process of having your source code repository as a single source of truth for Continous Integration and Continuous Delivery. The below code is a GitHub actions example to create topics in Kafka running on Confluent Cloud with GitHub actions.

```
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner and the image on which the job will run.
    runs-on: ubuntu-latest
    container: purbon/kafka-topology-builder:latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v2

      # Creates a dev.properties file with all the config required to establish a connection with Confluent Cloud
      - name: Create a properties file
        run: |
          ./build-properties-file.sh > dev.properties
          julie-ops-cli.sh --broker $DEV_BOOTSTRAP_SERVER --clientConfig dev.properties --topology descriptor-only-topics.yaml

        # Environment variables being used in the dev.properties file.
        env:
          DEV_BOOTSTRAP_SERVER: ${{ secrets.DEV_BOOTSTRAP_SERVER }}
          DEV_SASL_USERNAME: ${{ secrets.DEV_SASL_USERNAME }}
          DEV_SASL_PASSWORD: ${{ secrets.DEV_SASL_PASSWORD }}
          DEV_SCHEMA_REGISTRY_URL: ${{ secrets.DEV_SCHEMA_REGISTRY_URL }}
          DEV_SR_API_KEY: ${{ secrets.DEV_SR_API_KEY }}
          DEV_SR_API_SECRET: ${{ secrets.DEV_SR_API_SECRET }}
```

The source code of the GitHub actions file can be viewed [here][github_actions_file]

[julie-ops]: https://julieops.readthedocs.io/en/latest/#
[Topics]: https://julieops.readthedocs.io/en/latest/futures/what-topic-management.html
[Handling schemas]: https://julieops.readthedocs.io/en/latest/futures/what-schema-management.html
[Access Control]: https://julieops.readthedocs.io/en/latest/futures/what-acl-management.html
[ksql artifacts]: https://julieops.readthedocs.io/en/latest/futures/what-ksql-management.html
[topologies]: https://julieops.readthedocs.io/en/latest/the-descriptor-files.html?highlight=topology
[github_actions_file]: https://github.com/Platformatory/kafka-cd-julie/blob/dev/.github/workflows/dev.yml
