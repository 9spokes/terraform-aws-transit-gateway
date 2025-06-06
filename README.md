

<!-- markdownlint-disable -->
<a href="https://cpco.io/homepage"><img src="https://github.com/cloudposse/terraform-aws-transit-gateway/blob/main/.github/banner.png?raw=true" alt="Project Banner"/></a><br/>
    <p align="right">
<a href="https://github.com/cloudposse/terraform-aws-transit-gateway/releases/latest"><img src="https://img.shields.io/github/release/cloudposse/terraform-aws-transit-gateway.svg?style=for-the-badge" alt="Latest Release"/></a><a href="https://github.com/cloudposse/terraform-aws-transit-gateway/commits"><img src="https://img.shields.io/github/last-commit/cloudposse/terraform-aws-transit-gateway.svg?style=for-the-badge" alt="Last Updated"/></a><a href="https://cloudposse.com/slack"><img src="https://slack.cloudposse.com/for-the-badge.svg" alt="Slack Community"/></a></p>
<!-- markdownlint-restore -->

<!--




  ** DO NOT EDIT THIS FILE
  **
  ** This file was automatically generated by the `cloudposse/build-harness`.
  ** 1) Make all changes to `README.yaml`
  ** 2) Run `make init` (you only need to do this once)
  ** 3) Run`make readme` to rebuild this file.
  **
  ** (We maintain HUNDREDS of open source projects. This is how we maintain our sanity.)
  **





-->

Terraform module to provision:

- [AWS Transit Gateway](https://aws.amazon.com/transit-gateway/)
- [AWS Resource Access Manager (AWS RAM)](https://docs.aws.amazon.com/ram/latest/userguide/what-is.html) Resource Share to share the Transit Gateway with
  the Organization or another AWS Account (configurable via the variables `ram_resource_share_enabled` and `ram_principals`)
- [Transit Gateway route table](https://docs.aws.amazon.com/vpc/latest/tgw/tgw-route-tables.html)
- [Transit Gateway VPC attachments](https://docs.aws.amazon.com/vpc/latest/tgw/tgw-vpc-attachments.html) to connect multiple VPCs via the Transit Gateway
- Transit Gateway route table propagations to create propagated routes and allow traffic from the Transit Gateway to the VPC attachments
- Transit Gateway route table associations to allow traffic from the VPC attachments to the Transit Gateway
- Transit Gateway static routes (static routes have a higher precedence than propagated routes)
- Subnet routes to route traffic from the subnets in each VPC to the other Transit Gateway VPC attachments


> [!TIP]
> #### 👽 Use Atmos with Terraform
> Cloud Posse uses [`atmos`](https://atmos.tools) to easily orchestrate multiple environments using Terraform. <br/>
> Works with [Github Actions](https://atmos.tools/integrations/github-actions/), [Atlantis](https://atmos.tools/integrations/atlantis), or [Spacelift](https://atmos.tools/integrations/spacelift).
>
> <details>
> <summary><strong>Watch demo of using Atmos with Terraform</strong></summary>
> <img src="https://github.com/cloudposse/atmos/blob/main/docs/demo.gif?raw=true"/><br/>
> <i>Example of running <a href="https://atmos.tools"><code>atmos</code></a> to manage infrastructure from our <a href="https://atmos.tools/quick-start/">Quick Start</a> tutorial.</i>
> </detalis>


## Introduction

This module is configurable via the variable `transit_gateway_config` - see [usage](#usage) and [examples](#examples) below.

The variable `transit_gateway_config` is a map of environment names (e.g. `prod`, `staging`, `dev`) to the environment configurations.

Each environment configuration contains the following fields:

  - `vpc_id` - The ID of the VPC for which to create a VPC attachment and route table associations and propagations.
  - `vpc_cidr` - VPC CIDR block.
  - `subnet_ids` - The IDs of the subnets in the VPC.
  - `static_routes` - A list of Transit Gateway static route configurations. Note that static routes have a higher precedence than propagated routes.
  - `subnet_route_table_ids` - The IDs of the subnet route tables. The route tables are used to add routes to allow traffix from the subnets in one VPC
    to the other VPC attachments.
  - `route_to` - A set of environment names to route traffic from the current environment to the specified environments.
    In the example below, in the `prod` environment we create subnet routes to route traffic from the `prod` subnets to the VPC attachments in the `staging` and `dev` environments.
    Specify either `route_to` or `route_to_cidr_blocks`. `route_to_cidr_blocks` supersedes `route_to`.
  - `route_to_cidr_blocks` - A set of VPC CIDR blocks to route traffic from the current environment to the specified VPC CIDR blocks.
    In the example below, in the `staging` environment we create subnet routes to route traffic from the `staging` subnets to the `dev` VPC CIDR.
    Specify either `route_to` or `route_to_cidr_blocks`. `route_to_cidr_blocks` supersedes `route_to`.
  - `transit_gateway_vpc_attachment_id` - An existing Transit Gateway Attachment ID. If provided, the module will use it instead of creating a new one.

You now have the option to have Terraform manage route table entries by key, whereas previously they were only managed by index. The advantage
of managing them by key is that if a route table ID or destination CIDR changes, only that entry is affected, whereas when managed by index,
all the entries after the first affected index may be destroyed and re-created at a different index. The reason this is left as an option,
with the default being to manage the entries by index, is that if you are creating the VPC or subnets at the same time you are creating
the Transit Gateway, then Terraform will not be able to generate the keys during the plan phase and the plan will fail with the error
`The "for_each" value depends on resource attributes that cannot be determined until apply...`. We recommend setting `route_keys_enabled` to
`true` unless you get this error, in which case you must leave it set to its default value of `false`.

__NOTE:__ This module requires Terraform 0.13 and newer since it uses [module expansion with `for_each`](https://www.hashicorp.com/blog/announcing-hashicorp-terraform-0-13/).


> [!TIP]
> #### Use Terraform Reference Architectures for AWS
>
> Use Cloud Posse's ready-to-go [terraform architecture blueprints](https://cloudposse.com/reference-architecture/) for AWS to get up and running quickly.
>
> ✅ We build it together with your team.<br/>
> ✅ Your team owns everything.<br/>
> ✅ 100% Open Source and backed by fanatical support.<br/>
>
> <a href="https://cpco.io/commercial-support?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-transit-gateway&utm_content=commercial_support"><img alt="Request Quote" src="https://img.shields.io/badge/request%20quote-success.svg?style=for-the-badge"/></a>
> <details><summary>📚 <strong>Learn More</strong></summary>
>
> <br/>
>
> Cloud Posse is the leading [**DevOps Accelerator**](https://cpco.io/commercial-support?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-transit-gateway&utm_content=commercial_support) for funded startups and enterprises.
>
> *Your team can operate like a pro today.*
>
> Ensure that your team succeeds by using Cloud Posse's proven process and turnkey blueprints. Plus, we stick around until you succeed.
> #### Day-0:  Your Foundation for Success
> - **Reference Architecture.** You'll get everything you need from the ground up built using 100% infrastructure as code.
> - **Deployment Strategy.** Adopt a proven deployment strategy with GitHub Actions, enabling automated, repeatable, and reliable software releases.
> - **Site Reliability Engineering.** Gain total visibility into your applications and services with Datadog, ensuring high availability and performance.
> - **Security Baseline.** Establish a secure environment from the start, with built-in governance, accountability, and comprehensive audit logs, safeguarding your operations.
> - **GitOps.** Empower your team to manage infrastructure changes confidently and efficiently through Pull Requests, leveraging the full power of GitHub Actions.
>
> <a href="https://cpco.io/commercial-support?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-transit-gateway&utm_content=commercial_support"><img alt="Request Quote" src="https://img.shields.io/badge/request%20quote-success.svg?style=for-the-badge"/></a>
>
> #### Day-2: Your Operational Mastery
> - **Training.** Equip your team with the knowledge and skills to confidently manage the infrastructure, ensuring long-term success and self-sufficiency.
> - **Support.** Benefit from a seamless communication over Slack with our experts, ensuring you have the support you need, whenever you need it.
> - **Troubleshooting.** Access expert assistance to quickly resolve any operational challenges, minimizing downtime and maintaining business continuity.
> - **Code Reviews.** Enhance your team’s code quality with our expert feedback, fostering continuous improvement and collaboration.
> - **Bug Fixes.** Rely on our team to troubleshoot and resolve any issues, ensuring your systems run smoothly.
> - **Migration Assistance.** Accelerate your migration process with our dedicated support, minimizing disruption and speeding up time-to-value.
> - **Customer Workshops.** Engage with our team in weekly workshops, gaining insights and strategies to continuously improve and innovate.
>
> <a href="https://cpco.io/commercial-support?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-transit-gateway&utm_content=commercial_support"><img alt="Request Quote" src="https://img.shields.io/badge/request%20quote-success.svg?style=for-the-badge"/></a>
> </details>


## Usage

Here's how to invoke this module in your projects:

```hcl
locals {
  transit_gateway_config = {
    prod = {
      vpc_id                            = module.vpc_prod.vpc_id
      vpc_cidr                          = module.vpc_prod.vpc_cidr_block
      subnet_ids                        = module.subnets_prod.private_subnet_ids
      subnet_route_table_ids            = module.subnets_prod.private_route_table_ids
      route_to                          = ["staging", "dev"]
      route_to_cidr_blocks              = null
      transit_gateway_vpc_attachment_id = null

      static_routes = [
        {
          blackhole              = true
          destination_cidr_block = "0.0.0.0/0"
        },
        {
          blackhole              = false
          destination_cidr_block = "172.16.1.0/24"
        }
      ]
    },

    staging = {
      vpc_id                            = module.vpc_staging.vpc_id
      vpc_cidr                          = module.vpc_staging.vpc_cidr_block
      subnet_ids                        = module.subnets_staging.private_subnet_ids
      subnet_route_table_ids            = module.subnets_staging.private_route_table_ids
      route_to                          = null
      route_to_cidr_blocks              = [module.vpc_dev.vpc_cidr_block]
      transit_gateway_vpc_attachment_id = null

      static_routes = [
        {
          blackhole              = false
          destination_cidr_block = "172.32.1.0/24"
        }
      ]
    },

    dev = {
      vpc_id                            = module.vpc_dev.vpc_id
      vpc_cidr                          = module.vpc_dev.vpc_cidr_block
      subnet_ids                        = module.subnets_dev.private_subnet_ids
      subnet_route_table_ids            = module.subnets_dev.private_route_table_ids
      route_to                          = null
      route_to_cidr_blocks              = null
      transit_gateway_vpc_attachment_id = null
      static_routes                     = null
    }
  }
}

module "transit_gateway" {
  source = "cloudposse/transit-gateway/aws"
  # Cloud Posse recommends pinning every module to a specific version
  # version     = "x.x.x"

  ram_resource_share_enabled = false
  config                     = local.transit_gateway_config

  context = module.this.context
}
```

> [!IMPORTANT]
> In Cloud Posse's examples, we avoid pinning modules to specific versions to prevent discrepancies between the documentation
> and the latest released versions. However, for your own projects, we strongly advise pinning each module to the exact version
> you're using. This practice ensures the stability of your infrastructure. Additionally, we recommend implementing a systematic
> approach for updating versions to avoid unexpected changes.





## Examples

Here is a working example of using this module:
- [`examples/complete`](examples/complete)

Here are automated tests for the complete example using [bats](https://github.com/bats-core/bats-core) and [Terratest](https://github.com/gruntwork-io/terratest) (which tests and deploys the example on AWS):
- [`test`](test)

Here is an example of using this module in a multi-account environment (with the Transit Gateway in one AWS account and all the VPC attachments and routes in different AWS accounts):
- [`examples/multi-account`](examples/multi-account)




<!-- markdownlint-disable -->
## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 1.3 |
| <a name="requirement_aws"></a> [aws](#requirement\_aws) | >= 4.4.0 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_aws"></a> [aws](#provider\_aws) | >= 4.4.0 |

## Modules

| Name | Source | Version |
|------|--------|---------|
| <a name="module_subnet_route"></a> [subnet\_route](#module\_subnet\_route) | ./modules/subnet_route | n/a |
| <a name="module_this"></a> [this](#module\_this) | cloudposse/label/null | 0.25.0 |
| <a name="module_transit_gateway_route"></a> [transit\_gateway\_route](#module\_transit\_gateway\_route) | ./modules/transit_gateway_route | n/a |

## Resources

| Name | Type |
|------|------|
| [aws_ec2_transit_gateway.default](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ec2_transit_gateway) | resource |
| [aws_ec2_transit_gateway_route_table.default](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ec2_transit_gateway_route_table) | resource |
| [aws_ec2_transit_gateway_route_table_association.default](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ec2_transit_gateway_route_table_association) | resource |
| [aws_ec2_transit_gateway_route_table_propagation.default](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ec2_transit_gateway_route_table_propagation) | resource |
| [aws_ec2_transit_gateway_vpc_attachment.default](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ec2_transit_gateway_vpc_attachment) | resource |
| [aws_ram_principal_association.default](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ram_principal_association) | resource |
| [aws_ram_resource_association.default](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ram_resource_association) | resource |
| [aws_ram_resource_share.default](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ram_resource_share) | resource |
| [aws_ec2_transit_gateway.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/ec2_transit_gateway) | data source |
| [aws_organizations_organization.default](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/organizations_organization) | data source |
| [aws_vpc.default](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/vpc) | data source |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_additional_tag_map"></a> [additional\_tag\_map](#input\_additional\_tag\_map) | Additional key-value pairs to add to each map in `tags_as_list_of_maps`. Not added to `tags` or `id`.<br/>This is for some rare cases where resources want additional configuration of tags<br/>and therefore take a list of maps with tag key, value, and additional configuration. | `map(string)` | `{}` | no |
| <a name="input_allow_external_principals"></a> [allow\_external\_principals](#input\_allow\_external\_principals) | Indicates whether principals outside your organization can be associated with a resource share | `bool` | `false` | no |
| <a name="input_attributes"></a> [attributes](#input\_attributes) | ID element. Additional attributes (e.g. `workers` or `cluster`) to add to `id`,<br/>in the order they appear in the list. New attributes are appended to the<br/>end of the list. The elements of the list are joined by the `delimiter`<br/>and treated as a single ID element. | `list(string)` | `[]` | no |
| <a name="input_auto_accept_shared_attachments"></a> [auto\_accept\_shared\_attachments](#input\_auto\_accept\_shared\_attachments) | Whether resource attachment requests are automatically accepted. Valid values: `disable`, `enable`. Default value: `disable` | `string` | `"enable"` | no |
| <a name="input_config"></a> [config](#input\_config) | Configuration for VPC attachments, Transit Gateway routes, and subnet routes | <pre>map(object({<br/>    vpc_id                            = string<br/>    vpc_cidr                          = string<br/>    subnet_ids                        = set(string)<br/>    subnet_route_table_ids            = set(string)<br/>    route_to                          = set(string)<br/>    route_to_cidr_blocks              = set(string)<br/>    transit_gateway_vpc_attachment_id = string<br/>    static_routes = set(object({<br/>      blackhole              = bool<br/>      destination_cidr_block = string<br/>    }))<br/>  }))</pre> | `null` | no |
| <a name="input_context"></a> [context](#input\_context) | Single object for setting entire context at once.<br/>See description of individual variables for details.<br/>Leave string and numeric variables as `null` to use default value.<br/>Individual variable settings (non-null) override settings in context object,<br/>except for attributes, tags, and additional\_tag\_map, which are merged. | `any` | <pre>{<br/>  "additional_tag_map": {},<br/>  "attributes": [],<br/>  "delimiter": null,<br/>  "descriptor_formats": {},<br/>  "enabled": true,<br/>  "environment": null,<br/>  "id_length_limit": null,<br/>  "label_key_case": null,<br/>  "label_order": [],<br/>  "label_value_case": null,<br/>  "labels_as_tags": [<br/>    "unset"<br/>  ],<br/>  "name": null,<br/>  "namespace": null,<br/>  "regex_replace_chars": null,<br/>  "stage": null,<br/>  "tags": {},<br/>  "tenant": null<br/>}</pre> | no |
| <a name="input_create_transit_gateway"></a> [create\_transit\_gateway](#input\_create\_transit\_gateway) | Whether to create a Transit Gateway. If set to `false`, an existing Transit Gateway ID must be provided in the variable `existing_transit_gateway_id` | `bool` | `true` | no |
| <a name="input_create_transit_gateway_route_table"></a> [create\_transit\_gateway\_route\_table](#input\_create\_transit\_gateway\_route\_table) | Whether to create a Transit Gateway Route Table. If set to `false`, an existing Transit Gateway Route Table ID must be provided in the variable `existing_transit_gateway_route_table_id` | `bool` | `true` | no |
| <a name="input_create_transit_gateway_route_table_association_and_propagation"></a> [create\_transit\_gateway\_route\_table\_association\_and\_propagation](#input\_create\_transit\_gateway\_route\_table\_association\_and\_propagation) | Whether to create Transit Gateway Route Table associations and propagations | `bool` | `true` | no |
| <a name="input_create_transit_gateway_vpc_attachment"></a> [create\_transit\_gateway\_vpc\_attachment](#input\_create\_transit\_gateway\_vpc\_attachment) | Whether to create Transit Gateway VPC Attachments | `bool` | `true` | no |
| <a name="input_default_route_table_association"></a> [default\_route\_table\_association](#input\_default\_route\_table\_association) | Whether resource attachments are automatically associated with the default association route table. Valid values: `disable`, `enable`. Default value: `disable` | `string` | `"disable"` | no |
| <a name="input_default_route_table_propagation"></a> [default\_route\_table\_propagation](#input\_default\_route\_table\_propagation) | Whether resource attachments automatically propagate routes to the default propagation route table. Valid values: `disable`, `enable`. Default value: `disable` | `string` | `"disable"` | no |
| <a name="input_delimiter"></a> [delimiter](#input\_delimiter) | Delimiter to be used between ID elements.<br/>Defaults to `-` (hyphen). Set to `""` to use no delimiter at all. | `string` | `null` | no |
| <a name="input_descriptor_formats"></a> [descriptor\_formats](#input\_descriptor\_formats) | Describe additional descriptors to be output in the `descriptors` output map.<br/>Map of maps. Keys are names of descriptors. Values are maps of the form<br/>`{<br/>   format = string<br/>   labels = list(string)<br/>}`<br/>(Type is `any` so the map values can later be enhanced to provide additional options.)<br/>`format` is a Terraform format string to be passed to the `format()` function.<br/>`labels` is a list of labels, in order, to pass to `format()` function.<br/>Label values will be normalized before being passed to `format()` so they will be<br/>identical to how they appear in `id`.<br/>Default is `{}` (`descriptors` output will be empty). | `any` | `{}` | no |
| <a name="input_dns_support"></a> [dns\_support](#input\_dns\_support) | Whether resource attachments automatically propagate routes to the default propagation route table. Valid values: `disable`, `enable`. Default value: `enable` | `string` | `"enable"` | no |
| <a name="input_enabled"></a> [enabled](#input\_enabled) | Set to false to prevent the module from creating any resources | `bool` | `null` | no |
| <a name="input_environment"></a> [environment](#input\_environment) | ID element. Usually used for region e.g. 'uw2', 'us-west-2', OR role 'prod', 'staging', 'dev', 'UAT' | `string` | `null` | no |
| <a name="input_existing_transit_gateway_id"></a> [existing\_transit\_gateway\_id](#input\_existing\_transit\_gateway\_id) | Existing Transit Gateway ID. If provided, the module will not create a Transit Gateway but instead will use the existing one | `string` | `null` | no |
| <a name="input_existing_transit_gateway_route_table_id"></a> [existing\_transit\_gateway\_route\_table\_id](#input\_existing\_transit\_gateway\_route\_table\_id) | Existing Transit Gateway Route Table ID. If provided, the module will not create a Transit Gateway Route Table but instead will use the existing one | `string` | `null` | no |
| <a name="input_id_length_limit"></a> [id\_length\_limit](#input\_id\_length\_limit) | Limit `id` to this many characters (minimum 6).<br/>Set to `0` for unlimited length.<br/>Set to `null` for keep the existing setting, which defaults to `0`.<br/>Does not affect `id_full`. | `number` | `null` | no |
| <a name="input_label_key_case"></a> [label\_key\_case](#input\_label\_key\_case) | Controls the letter case of the `tags` keys (label names) for tags generated by this module.<br/>Does not affect keys of tags passed in via the `tags` input.<br/>Possible values: `lower`, `title`, `upper`.<br/>Default value: `title`. | `string` | `null` | no |
| <a name="input_label_order"></a> [label\_order](#input\_label\_order) | The order in which the labels (ID elements) appear in the `id`.<br/>Defaults to ["namespace", "environment", "stage", "name", "attributes"].<br/>You can omit any of the 6 labels ("tenant" is the 6th), but at least one must be present. | `list(string)` | `null` | no |
| <a name="input_label_value_case"></a> [label\_value\_case](#input\_label\_value\_case) | Controls the letter case of ID elements (labels) as included in `id`,<br/>set as tag values, and output by this module individually.<br/>Does not affect values of tags passed in via the `tags` input.<br/>Possible values: `lower`, `title`, `upper` and `none` (no transformation).<br/>Set this to `title` and set `delimiter` to `""` to yield Pascal Case IDs.<br/>Default value: `lower`. | `string` | `null` | no |
| <a name="input_labels_as_tags"></a> [labels\_as\_tags](#input\_labels\_as\_tags) | Set of labels (ID elements) to include as tags in the `tags` output.<br/>Default is to include all labels.<br/>Tags with empty values will not be included in the `tags` output.<br/>Set to `[]` to suppress all generated tags.<br/>**Notes:**<br/>  The value of the `name` tag, if included, will be the `id`, not the `name`.<br/>  Unlike other `null-label` inputs, the initial setting of `labels_as_tags` cannot be<br/>  changed in later chained modules. Attempts to change it will be silently ignored. | `set(string)` | <pre>[<br/>  "default"<br/>]</pre> | no |
| <a name="input_name"></a> [name](#input\_name) | ID element. Usually the component or solution name, e.g. 'app' or 'jenkins'.<br/>This is the only ID element not also included as a `tag`.<br/>The "name" tag is set to the full `id` string. There is no tag with the value of the `name` input. | `string` | `null` | no |
| <a name="input_namespace"></a> [namespace](#input\_namespace) | ID element. Usually an abbreviation of your organization name, e.g. 'eg' or 'cp', to help ensure generated IDs are globally unique | `string` | `null` | no |
| <a name="input_ram_principal"></a> [ram\_principal](#input\_ram\_principal) | DEPRECATED, please use ram\_principals instead.<br/><br/>The principal to associate with the resource share. Possible values are an<br/>AWS account ID, an Organization ARN, or an Organization Unit ARN. | `string` | `null` | no |
| <a name="input_ram_principals"></a> [ram\_principals](#input\_ram\_principals) | A list of principals to associate with the resource share. Possible values<br/>are:<br/><br/>* AWS account ID<br/>* Organization ARN<br/>* Organization Unit ARN<br/><br/>If this (and var.ram\_principal) is not provided and<br/>`ram_resource_share_enabled` is `true`, the Organization ARN will be used. | `list(string)` | `[]` | no |
| <a name="input_ram_resource_share_enabled"></a> [ram\_resource\_share\_enabled](#input\_ram\_resource\_share\_enabled) | Whether to enable sharing the Transit Gateway with the Organization using Resource Access Manager (RAM) | `bool` | `false` | no |
| <a name="input_regex_replace_chars"></a> [regex\_replace\_chars](#input\_regex\_replace\_chars) | Terraform regular expression (regex) string.<br/>Characters matching the regex will be removed from the ID elements.<br/>If not set, `"/[^a-zA-Z0-9-]/"` is used to remove all characters other than hyphens, letters and digits. | `string` | `null` | no |
| <a name="input_route_keys_enabled"></a> [route\_keys\_enabled](#input\_route\_keys\_enabled) | If true, Terraform will use keys to label routes, preventing unnecessary changes,<br/>but this requires that the VPCs and subnets already exist before using this module.<br/>If false, Terraform will use numbers to label routes, and a single change may<br/>cascade to a long list of changes because the index or order has changed, but<br/>this will work when the `true` setting generates the error `The "for_each" value depends on resource attributes...` | `bool` | `false` | no |
| <a name="input_route_timeouts"></a> [route\_timeouts](#input\_route\_timeouts) | aws\_route resource timeouts | <pre>object({<br/>    create = optional(string),<br/>    delete = optional(string),<br/>    update = optional(string)<br/>  })</pre> | `{}` | no |
| <a name="input_stage"></a> [stage](#input\_stage) | ID element. Usually used to indicate role, e.g. 'prod', 'staging', 'source', 'build', 'test', 'deploy', 'release' | `string` | `null` | no |
| <a name="input_tags"></a> [tags](#input\_tags) | Additional tags (e.g. `{'BusinessUnit': 'XYZ'}`).<br/>Neither the tag keys nor the tag values will be modified by this module. | `map(string)` | `{}` | no |
| <a name="input_tenant"></a> [tenant](#input\_tenant) | ID element \_(Rarely used, not included by default)\_. A customer identifier, indicating who this instance of a resource is for | `string` | `null` | no |
| <a name="input_transit_gateway_cidr_blocks"></a> [transit\_gateway\_cidr\_blocks](#input\_transit\_gateway\_cidr\_blocks) | The list of associated CIDR blocks. It can contain up to 1 IPv4 CIDR block<br/>of size up to /24 and up to one IPv6 CIDR block of size up to /64. The IPv4<br/>block must not be from range 169.254.0.0/16. | `list(string)` | `null` | no |
| <a name="input_transit_gateway_description"></a> [transit\_gateway\_description](#input\_transit\_gateway\_description) | Transit Gateway description. If not provided, one will be automatically generated. | `string` | `""` | no |
| <a name="input_vpc_attachment_appliance_mode_support"></a> [vpc\_attachment\_appliance\_mode\_support](#input\_vpc\_attachment\_appliance\_mode\_support) | Whether Appliance Mode support is enabled. If enabled, a traffic flow between a source and destination uses the same Availability Zone for the VPC attachment for the lifetime of that flow. Valid values: `disable`, `enable` | `string` | `"disable"` | no |
| <a name="input_vpc_attachment_dns_support"></a> [vpc\_attachment\_dns\_support](#input\_vpc\_attachment\_dns\_support) | Whether resource attachments automatically propagate routes to the default propagation route table. Valid values: `disable`, `enable`. Default value: `enable` | `string` | `"enable"` | no |
| <a name="input_vpc_attachment_ipv6_support"></a> [vpc\_attachment\_ipv6\_support](#input\_vpc\_attachment\_ipv6\_support) | Whether resource attachments automatically propagate routes to the default propagation route table. Valid values: `disable`, `enable`. Default value: `enable` | `string` | `"disable"` | no |
| <a name="input_vpn_ecmp_support"></a> [vpn\_ecmp\_support](#input\_vpn\_ecmp\_support) | Whether resource attachments automatically propagate routes to the default propagation route table. Valid values: `disable`, `enable`. Default value: `enable` | `string` | `"enable"` | no |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_ram_resource_share_id"></a> [ram\_resource\_share\_id](#output\_ram\_resource\_share\_id) | RAM resource share ID |
| <a name="output_subnet_route_ids"></a> [subnet\_route\_ids](#output\_subnet\_route\_ids) | Subnet route identifiers combined with destinations |
| <a name="output_transit_gateway_arn"></a> [transit\_gateway\_arn](#output\_transit\_gateway\_arn) | Transit Gateway ARN |
| <a name="output_transit_gateway_association_default_route_table_id"></a> [transit\_gateway\_association\_default\_route\_table\_id](#output\_transit\_gateway\_association\_default\_route\_table\_id) | Transit Gateway association default route table ID |
| <a name="output_transit_gateway_id"></a> [transit\_gateway\_id](#output\_transit\_gateway\_id) | Transit Gateway ID |
| <a name="output_transit_gateway_propagation_default_route_table_id"></a> [transit\_gateway\_propagation\_default\_route\_table\_id](#output\_transit\_gateway\_propagation\_default\_route\_table\_id) | Transit Gateway propagation default route table ID |
| <a name="output_transit_gateway_route_ids"></a> [transit\_gateway\_route\_ids](#output\_transit\_gateway\_route\_ids) | Transit Gateway route identifiers combined with destinations |
| <a name="output_transit_gateway_route_table_id"></a> [transit\_gateway\_route\_table\_id](#output\_transit\_gateway\_route\_table\_id) | Transit Gateway route table ID |
| <a name="output_transit_gateway_vpc_attachment_ids"></a> [transit\_gateway\_vpc\_attachment\_ids](#output\_transit\_gateway\_vpc\_attachment\_ids) | Transit Gateway VPC attachment IDs |
<!-- markdownlint-restore -->







## Related Projects

Check out these related projects.

- [terraform-null-label](https://github.com/cloudposse/terraform-null-label) - Terraform module designed to generate consistent names and tags for resources. Use terraform-null-label to implement a strict naming convention
- [terraform-aws-vpc](https://github.com/cloudposse/terraform-aws-vpc) - Terraform Module that defines a VPC with public/private subnets across multiple AZs with Internet Gateways
- [terraform-aws-vpc-peering](https://github.com/cloudposse/terraform-aws-vpc-peering) - Terraform module to create a peering connection between two VPCs
- [terraform-aws-kops-vpc-peering](https://github.com/cloudposse/terraform-aws-kops-vpc-peering) - Terraform module to create a peering connection between a backing services VPC and a VPC created by Kops
- [terraform-aws-dynamic-subnets](https://github.com/cloudposse/terraform-aws-dynamic-subnets) - Terraform module for public and private subnets provisioning in an existing VPC
- [terraform-aws-multi-az-subnets](https://github.com/cloudposse/terraform-aws-multi-az-subnets) - Terraform module for multi-AZ public and private subnets provisioning
- [terraform-aws-named-subnets](https://github.com/cloudposse/terraform-aws-named-subnets) - Terraform module for named subnets provisioning


## References

For additional context, refer to some of these links.

- [Terraform Standard Module Structure](https://www.terraform.io/docs/modules/index.html#standard-module-structure) - HashiCorp's standard module structure is a file and directory layout we recommend for reusable modules distributed in separate repositories.
- [Terraform Module Requirements](https://www.terraform.io/docs/registry/modules/publish.html#requirements) - HashiCorp's guidance on all the requirements for publishing a module. Meeting the requirements for publishing a module is extremely easy.
- [Terraform `random_integer` Resource](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/integer) - The resource random_integer generates random values from a given range, described by the min and max attributes of a given resource.
- [Terraform Version Pinning](https://www.terraform.io/docs/configuration/terraform.html#specifying-a-required-terraform-version) - The required_version setting can be used to constrain which versions of the Terraform CLI can be used with your configuration




## ✨ Contributing

This project is under active development, and we encourage contributions from our community.



Many thanks to our outstanding contributors:

<a href="https://github.com/cloudposse/terraform-aws-transit-gateway/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=cloudposse/terraform-aws-transit-gateway&max=24" />
</a>

For 🐛 bug reports & feature requests, please use the [issue tracker](https://github.com/cloudposse/terraform-aws-transit-gateway/issues).

In general, PRs are welcome. We follow the typical "fork-and-pull" Git workflow.
 1. Review our [Code of Conduct](https://github.com/cloudposse/terraform-aws-transit-gateway/?tab=coc-ov-file#code-of-conduct) and [Contributor Guidelines](https://github.com/cloudposse/.github/blob/main/CONTRIBUTING.md).
 2. **Fork** the repo on GitHub
 3. **Clone** the project to your own machine
 4. **Commit** changes to your own branch
 5. **Push** your work back up to your fork
 6. Submit a **Pull Request** so that we can review your changes

**NOTE:** Be sure to merge the latest changes from "upstream" before making a pull request!

### 🌎 Slack Community

Join our [Open Source Community](https://cpco.io/slack?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-transit-gateway&utm_content=slack) on Slack. It's **FREE** for everyone! Our "SweetOps" community is where you get to talk with others who share a similar vision for how to rollout and manage infrastructure. This is the best place to talk shop, ask questions, solicit feedback, and work together as a community to build totally *sweet* infrastructure.

### 📰 Newsletter

Sign up for [our newsletter](https://cpco.io/newsletter?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-transit-gateway&utm_content=newsletter) and join 3,000+ DevOps engineers, CTOs, and founders who get insider access to the latest DevOps trends, so you can always stay in the know.
Dropped straight into your Inbox every week — and usually a 5-minute read.

### 📆 Office Hours <a href="https://cloudposse.com/office-hours?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-transit-gateway&utm_content=office_hours"><img src="https://img.cloudposse.com/fit-in/200x200/https://cloudposse.com/wp-content/uploads/2019/08/Powered-by-Zoom.png" align="right" /></a>

[Join us every Wednesday via Zoom](https://cloudposse.com/office-hours?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-transit-gateway&utm_content=office_hours) for your weekly dose of insider DevOps trends, AWS news and Terraform insights, all sourced from our SweetOps community, plus a _live Q&A_ that you can’t find anywhere else.
It's **FREE** for everyone!
## License

<a href="https://opensource.org/licenses/Apache-2.0"><img src="https://img.shields.io/badge/License-Apache%202.0-blue.svg?style=for-the-badge" alt="License"></a>

<details>
<summary>Preamble to the Apache License, Version 2.0</summary>
<br/>
<br/>

Complete license is available in the [`LICENSE`](LICENSE) file.

```text
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
```
</details>

## Trademarks

All other trademarks referenced herein are the property of their respective owners.


## Copyrights

Copyright © 2020-2025 [Cloud Posse, LLC](https://cloudposse.com)



<a href="https://cloudposse.com/readme/footer/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-transit-gateway&utm_content=readme_footer_link"><img alt="README footer" src="https://cloudposse.com/readme/footer/img"/></a>

<img alt="Beacon" width="0" src="https://ga-beacon.cloudposse.com/UA-76589703-4/cloudposse/terraform-aws-transit-gateway?pixel&cs=github&cm=readme&an=terraform-aws-transit-gateway"/>
