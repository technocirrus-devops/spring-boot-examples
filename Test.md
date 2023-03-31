<!-- BEGIN_TF_DOCS -->
## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_aws"></a> [aws](#requirement\_aws) | >= 3.36 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_aws"></a> [aws](#provider\_aws) | >= 3.36 |

## Modules

No modules.

## Resources

| Name | Type |
|------|------|
| [aws_lb.nlb](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb) | resource |
| [aws_lb_listener.listener](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb_listener) | resource |
| [aws_lb_target_group.target_group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb_target_group) | resource |


## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_nlb_name"></a> [nlb\_name](#input\_nlb\_name) | network load balancer name | `string` | `""` | no |
| <a name="input_load_balancer_type"></a> [load\_balancer\_type](#input\_load\_balancer\_type) | type of load balancer | `string` | `"network"` | no |
| <a name="input_subnets"></a> [subnets](#input\_subnets) | subnets for load balancer | `list(any)` | `""` | yes |
| <a name="input_internal"></a> [internal](#input\_internal) | True if load balancer is internal | `bool` | `true` | no |
| <a name="input_cross_zone_lb"></a> [cross\_zone\_lb](#input\_cross\_zone\_lb) | enable or disable cross zone load balancing | `bool` | `""` | no |
| <a name="input_nlb_target"></a> [nlb\_target](#input\_nlb\_target) | This is map variable for target group | `list(map(any))` | `""` | no |
| <a name="input_nlb_listener"></a> [nlb\_listener](#input\_nlb\_listener) | map variable for lb listener | `map(object({
    type            = string
    port            = number
    protocol        = string
    tg_index_number = number
    ssl_policy      = string
    certificate_arn = string
  }))` | `""` | no |
| <a name="input_vpc_id"></a> [vpc\_id](#input\_vpc\_id) | VPC Id | `string` | `""` | no |
| <a name="input_tag_name"></a> [tag\_name](#input\_tag\_name) | Name tag for NLB resources | `string` | `""` | no |


## Outputs

| Name | Description |
|------|-------------|
| <a name="output_lb_arn"></a> [lb\_arn](#output\_lb\_arn) | n/a |
| <a name="output_lb_dns_name"></a> [lb\_dns\_name](#output\_lb\_dns\_name) | n/a |
<!-- END_TF_DOCS -->
