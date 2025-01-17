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
| [aws_route53_zone.route53_private_zone](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route53_zone) | resource |
| [aws_route53_record.simple](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route53_record) | resource |
| [aws_route53_record.alias](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route53_record) | resource |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_zone_name"></a> [zone\_name](#input\_zone\_name) | Route53 Zone Name | `string` | `` | yes |
| <a name="input_vpc_id"></a> [vpc\_id](#input\_vpc\_id) | VPC ID | `string` | `""` | yes |
| <a name="input_route53_simple_records"></a> [route53\_simple\_records](#input\_route53\_simple\_records) | Route 53 non alias records | `list(object({}))` | `""` | no |
| <a name="input_route53_alias_records"></a> [route53\_alias\_records](#input\_route53\_alias\_records) | Route53 Alias records | `list(object({}))` | `""` | no |
| <a name="input_tag_name"></a> [tag\_name](#input\_tag\_name) | Name tag for Route53 Zone | `string` | `""` | no |
| <a name="input_local_secondary_indexes"></a> [local\_secondary\_indexes](#input\_local\_secondary\_indexes) | n/a | `list(object({ `<br>  `name = string `<br>  `range_key = string` <br>  `projection_type = string` <br>  `non_key_attributes = list(string)`<br>  `}))` | `""` | no |


## Outputs

| Name | Description |
|------|-------------|
| <a name="output_route53_zone_id"></a> [route53\_zone\_id](#output\_route53\_zone\_id) | Rout53 zone ID |
<!-- END_TF_DOCS -->


| Column 1 | Column 2 |
| -------- | -------- |
| Row 1, Cell 1<br>Row 1, Cell 1 continued | Row 1, Cell 2 |
| Row 2, Cell 1<br>Row 2, Cell 1 continued<br>Row 2, Cell 1 continued further | Row 2, Cell 2 |

[Link to documentation](./spring-boot-2-jdbc-with-h2/src)
