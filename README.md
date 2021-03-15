<!-- note marker start -->
**NOTE**: This repo contains only the documentation for the private BoltsOps Pro repo code.
Original file: https://github.com/boltopspro/s3-secure/blob/master/README.md
The docs are publish so they are available for interested customers.
For access to the source code, you must be a paying BoltOps Pro subscriber.
If are interested, you can contact us at contact@boltops.com or https://www.boltops.com

<!-- note marker end -->

# S3 Secure Blueprint

This blueprint auto-remediates and hardens newly created s3 buckets.  When new buckets are created, a Lambda function is run that applies security controls. This ensures all buckets are compliant.  This CloudFormation template can be used as-is or used as an example to tailor for your needs.

* Enables Encryption on S3 Buckets
* Enables Bucket Policy that enforces SSL Transport

The blueprint also supports:

* Optionally [VpcConfig](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-function.html#cfn-lambda-function-vpcconfig). Variables can be used to set different values for multiple environments.
* Supports [Lambda X-Ray tracing](https://docs.aws.amazon.com/lambda/latest/dg/lambda-x-ray.html)
* Can customize [EventPattern](https://docs.aws.amazon.com/eventbridge/latest/userguide/aws-events.html).

## Usage

1. Add blueprint to Gemfile
2. Configure: configs/s3-secure values
3. Deploy blueprint

## Configure

First you want to configure the `configs/s3-secure` [config files](https://lono.cloud/docs/core/configs/).  You can use [lono seed](https://lono.cloud/reference/lono-seed/) to configure starter values quickly.

    LONO_ENV=development lono seed s3-secure

For additional environments:

    LONO_ENV=production  lono seed s3-secure

The generated files in `config/s3-secure` folder look something like this:

    configs/s3-secure/
    └── variables
        ├── development.rb
        └── production.rb

## Deploy

Use the [lono cfn deploy](https://lono.cloud/reference/lono-cfn-deploy/) command to deploy. Example:

    LONO_ENV=development lono cfn deploy s3-secure --sure
    LONO_ENV=production  lono cfn deploy s3-secure --sure

If you are using One AWS Account, use these commands instead: [One Account](docs/one-account.md).

## Configure Details: Event Pattern

The default behavior listens for the S3 Create Bucket event. You can adjust the default `@event_pattern` if needed:

configs/s3-secure/variables/development.rb:

```ruby
@event_pattern = {
  source: ["aws.s3"],
  "detail-type": ["AWS API Call via CloudTrail"],
  detail: {
    eventName: ["CreateBucket"],
    eventSource: ["s3.amazonaws.com"],
  }
}
```

Here are the [EventPattern](https://docs.aws.amazon.com/eventbridge/latest/userguide/aws-events.html) docs.

### Lambda Function in VPC

To configure the Lambda function with VpcConfig set `@subnet_ids` and either `@vpc_id` or `@security_group_ids`.

* When the `@vpc_id` is set, the template creates a managed security group for you and the Lambda function is configured to use that security group.
* When `@security_group_ids` is set, the Lambda function will use those existing security groups.
* The subnet must be a private subnet with configured with a NAT.

Here's an example of the managed security group.

configs/lambda/variables/development.rb:

```ruby
@subnet_ids = ["subnet-111"]
@vpc_id = "vpc-111"
```

For Lambda VPC to work, the subnet must be a private subnet configured with a NAT.

Note, Lambda functions configured with VPCs may take much longer to deploy, typically 30-45 minutes. This is because Lambda creates and attaches an ENI to the Lambda function to make the VPC feature possible. If the function is deleted or updated, requiring replacement, the ENI takes 30-45m to be removed. Because of this, it is recommended to write code for your Lambda function code without the VpcConfig first. Get it working and then add VpcConfig at the end.

### X-Ray Tracing

Lambda X-Ray tracing is set to `Active` by default. You can disable this by setting `@tracing_config_mode = false`. Example:

configs/lambda/variables/development.rb:

```ruby
@tracing_config_mode = false
```

You can also change the mode with the same `@tracing_config_mode` variable:

```ruby
@tracing_config_mode = "Active" # or "PassThrough"
```

