---
layout: post
title: "Multi account Terraform GitOps for AWS"
date: 2021-10-06
permalink: "/tech/multi-account-terraform"
---

As discussed in the earlier blog post, Terraform is idempotent. You cannot replicate the same set of Infrastructure. But that wouldn't be the case always. At times, we might need to replicate our infrastructure when working with different environments. That's when Terraform workspace comes in handy. It helps us to create multiple infrastructures with the same configuration but in different workspaces. 

<!--more-->

By default, Terraform initializes with a single workspace called `default`. Also, we cannot delete this workspace.  Workspaces can be managed with the command `terraform workspace <COMMAND_NAME>`

We create a new workspace with

```
terraform workspace new <WORKSPACE_NAME>
```

We can also conditionally configure our infrastructure based on workspace name. For example,

```
resource "aws_instance" "example" {
  count = "${terraform.workspace == "production" ? 10 : 1}"

  # ... other arguments
}
````

In the above example, when the workspace is production, we provision 10 instances whereas for a non-production workspace we provision lesser instances.

### When to use workspaces

We use multiple workspaces to provision a parallel, distinct copy of a set of infrastructure to test a set of changes before modifying the infrastructure in a production environment. For example, a developer working on a complex set of infrastructure changes might create a new temporary workspace like dev or test to freely experiment with changes without affecting the production workspace.

### Deploy using GitOps

Everything in the modern world is deployed with Continous Deployment. Let's see how to deploy a terraform with GitOps. The below is an example GitHub actions to deploy any Terraform script to AWS.

```
jobs:
  terraform:
    name: 'Terraform'
    runs-on: ubuntu-latest

    steps:
    # Checkout the repository to the GitHub Actions runner
    - name: Checkout
      uses: actions/checkout@v2

    # Install the latest version of Terraform CLI and configure the Terraform CLI configuration file with a Terraform Cloud user API token
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v1
      with:
        cli_config_credentials_token: ${{ secrets.TERRAFORM_API_TOKEN }}

    # Initialize a new or existing Terraform working directory by creating initial files, loading any remote state, downloading modules, etc.
    - name: Terraform Init
      run: terraform init

    # Generates an execution plan for Terraform
    - name: Terraform Plan
      run: terraform plan
      // AWS Secrets for Terraform
      env:
        AWS_ACCESS_ID: ${{ secrets.AWS_ACCESS_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```
