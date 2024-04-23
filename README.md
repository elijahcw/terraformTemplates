# Introduction 
This repo contains azure pipeline templates to be referenced by other pipelines. These common states can be re-used and updated
independently of the pipelines that consume them.

# Terraform Templates
The `terraform-templates` directory contains templates specific to operations around terraform/terragrunt.
These templates are meant to be consumed together to create a full terraform deployment pipeline, however they can also be consumed as individual stages if necessary.

## Prerequisites
There a few prerequisites required in order to utilize these templates for your deployment pipelines:

* Certain variables must be made available to the pipeline:
  - Azure Service Principal Credentials including `AzureClientId`, `AzureClientSecret`, `AzureSubscriptionId`, `AzureTenantId`.
  - Terraform Enterprise Team Token (`TFETeamToken`)
  - ssh configuration (WIP)

* To utilize these terraform templates, you must be following a gitops pattern of deployment and your repo structure must be laid in a similar pattern as below: 
  ```
  ├── environments
  │   ├── dev
  │   │   └── terragrunt.hcl
  │   ├── model
  │   │   └── terragrunt.hcl
  │   └── terragrunt.hcl
  └── terraform
      ├── main.tf
      ├── variables.tf
      └── versions.tf
  ```
  The key takeaways here are an `environments` folder that contains the terragrunt for each environment that you wish to deploy to (dev, model, prod etc) as well as a `terraform` directory that contains the terraform you wish to deploy for each environment.
  These repos are designed for GitOps deployment and thus each environment that you wish to deploy to should have a corresponding branch to reflect the state of that environment. An example GitOps repo that follows this directory structure and consumes these templates can be found [here](https://dev.azure.com/AIZ-GT/GT.ICS-FIT/_git/deploy-iac-sdlc-reference)

## Validate (validate.yml)
This template runs various checks to make sure that your terraform code is properly validated.

The following checks are run against your terraform code:
  - terraform validate - Verifies whether a configuration is syntactically valid and internally consistent
  - terraform format - Checks if the terraform is formatted properly
  - [tflint](https://github.com/terraform-linters/tflint) - Lints terraform to enforce best practices, catch possible errors, etc
  - terraform plan - Generates an execution plan for Terraform

### Parameters
The validate template takes in the following parameters
  - `terraform_source_path` - Path to the directory that contains terraform code. Default value is `terraform`
  - `dryrun` - If false, executes the stage laid out in the template. Set to true only for template validation. Default value is `false`
  - `terraform_version` - Version of terraform to use within the stage template. Default value is `0.13.4`
  - `environments` - Environments to validate. All environments that are defined in your `environments` directory should be passed in here. Default value is `{}` (no environments)
  
  
## Plan (plan.yml)
This template generates an execution plan for your terraform while providing additional feedback and direction through pipeline notifications and extensions.

### Parameters 
The plan template takes in the following parameters
  - `terraform_source_path` - Path to the directory that contains terraform code. Default value is `terraform`
  - `dryrun` - If false, executes the stage laid out in the template. Set to true only for template validation. Default value is `false`
  - `terraform_version` - Version of terraform to use within the stage template. Default value is `0.13.4`
  - `environments` - Environments to generate a plan against. All environments that are defined in your `environments` directory should be passed in here. Default value is `{}` (no environments). Important to note that the stage defined in this template
    will only be executed on the corresponding environment branch (ie the plan for the `dev` environment will only be generated when the pipeline is executed from the `dev` branch, etc)
  - `authorizeDestructionOfResources` - Authorize the Destruction of Terraform provisioned resources. If your terraform plan includes the destruction of resources then this value must be set to true. Default value is 'false'
  - `usingTFEWorkspace` - If true, Terraform Enterprise specific feedback is given in the pipeline. Set to false if not using terraform enterprise workspaces.

## Apply (apply.yml)
This template provisions the resources in a terraform stack via `terragrunt apply`

### Parameters
The apply template takes in the following parameters
  - `terraform_source_path` - Path to the directory that contains terraform code. Default value is `terraform`
  - `dryrun` - If false, executes the stage laid out in the template. Set to true only for template validation. Default value is `false`
  - `terraform_version` - Version of terraform to use within the stage template. Default value is `0.13.4`
  - `environments` - Environments to generate a plan against. All environments that are defined in your `environments` directory should be passed in here. Default value is `{}` (no environments). Important to note that the stage defined in this template
    will only be executed on the corresponding environment branch (ie the plan for the `dev` environment will only be generated when the pipeline is executed from the `dev` branch, etc)
  - `authorizeDestructionOfResources` - Authorize the Destruction of Terraform provisioned resources. If your terraform plan includes the destruction of resources then this value must be set to true. Default value is 'false'
  - `usingTFEWorkspace` - If true, Terraform Enterprise specific feedback is given in the pipeline. Set to false if not using terraform enterprise workspaces.

# Usage
1. Reference this repository in your pipeline as a resource
``` yaml
resources:
  repositories:
    - repository: templates
      type: git
      name: templates-iac-pipeline-deployment
      ref: refs/tags/latest
```
2. Add each template you wish to use in the appropriate location
3. Add any parameters required by the template
## Example:
``` yaml
stages:
 - template:  terraform-templates/validate.yml@templates
   parameters:
     environments:
       - "dev"   
       - "model"
```
