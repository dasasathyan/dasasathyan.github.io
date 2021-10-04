---
layout: post
title: "Create AWS API Gateway with Swagger in Terraform"
date: 2021-10-03
---

API Gateway helps developers to create, manage and monitor APIs at any scale between a client and one or more backend services. Amazon Web Services has its own fully managed API Gateway service with which developers can manage their APIs. It also manages thousands of concurrent API calls, traffic management, CORS support, authorization and access control, throttling, monitoring, and API version management.

Terraform is an Infrastructure as a Code(IaC) tool. Read more about it [here](2021-10-03-gitops-in-kafka.md)

<!--more-->

APIs can have a lot of endpoints and each endpoint would have a lot of configurations like request payload, response, success and error response. Creating and configuring such an API using the AWS console would consume a lot of time. API Gateway also lets us do that just by importing the Swagger file to API Gateway. If there are multiple APIs, importing them using the console is again time-consuming.

An even simpler way is to use Terraform where you need to just pass the swagger file and run it. We can use Terraform's `aws_api_gateway_rest_api` resource to import an API with AWS API Gateway just by passing the swagger file as shown in the example below.

```
    resource "aws_api_gateway_rest_api" "api_name" {
        name = "websec-${terraform.workspace}"
        body = templatefile("<swagger_file>", {
            ENV_VARIABLE_KEY_1 = ENV_VARIABLE_VALUE_1
            ENV_VARIABLE_KEY_2 = ENV_VARIABLE_VALUE_2
        })
    }
```
