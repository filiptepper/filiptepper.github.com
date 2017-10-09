---
title: Auto scaling DynamoDB tables with Ruby's aws-sdk
layout: post
comments: true
---

Recently Amazon introduced [DynamoDB auto scaling](https://aws.amazon.com/blogs/aws/new-auto-scaling-for-amazon-dynamodb/), which we were very keen to try. Our current solution was flaky at best, and caused numerous issues with under- and overprovisioning.

Some of our DynamoDB tables are created dynamically - we use Ruby `aws-sdk` gem for that. To add autoscaling to already existing table follow this example, which sets desired capacity utilization to 75% for both reads and writes, with 100 as maximum capacity for reads, and 800 maximum capacity for writes.

```ruby
client = Aws::ApplicationAutoScaling::Client.new(
  region:            ENV['AUTOSCALING_AWS_REGION'],
  access_key_id:     ENV['AUTOSCALING_AWS_KEY'],
  secret_access_key: ENV['AUTOSCALING_AWS_SECRET']
)

# table name needs to prefixed with 'table/'
resource_id = 'table/TABLE_NAME'

target_options = {
  resource_id: resource_id,
  # replace '012345678901' with your account
  role_arn: 'arn:aws:iam::012345678901:role/service-role/DynamoDBAutoscaleRole',
  service_namespace: 'dynamodb'
}

# http://docs.aws.amazon.com/sdkforruby/api/Aws/ApplicationAutoScaling/Client.html#register_scalable_target-instance_method
client.register_scalable_target(
  target_options.merge(
    max_capacity: 100,
    min_capacity: 1,
    scalable_dimension: 'dynamodb:table:ReadCapacityUnits',
  )
)

client.register_scalable_target(
  target_options.merge(
    max_capacity: 800,
    min_capacity: 1,
    scalable_dimension: 'dynamodb:table:WriteCapacityUnits',
  )
)

policy_options = {
  service_namespace: 'dynamodb',
  resource_id: resource_id,
  policy_type: 'TargetTrackingScaling',
}

# http://docs.aws.amazon.com/sdkforruby/api/Aws/ApplicationAutoScaling/Client.html#put_scaling_policy-instance_method
client.put_scaling_policy(
  policy_options.merge(
    policy_name: 'dynamodb-read-autoscaling-TABLE_NAME',
    scalable_dimension: 'dynamodb:table:ReadCapacityUnits',
    target_tracking_scaling_policy_configuration: {
      target_value: 75.0,
      predefined_metric_specification: {
        predefined_metric_type: 'DynamoDBReadCapacityUtilization'
      },
      scale_out_cooldown: 1,
      scale_in_cooldown: 1
    }
  )
)

client.put_scaling_policy(
  policy_options.merge(
    policy_name: 'dynamodb-write-autoscaling-TABLE_NAME',
    scalable_dimension: 'dynamodb:table:WriteCapacityUnits',
    target_tracking_scaling_policy_configuration: {
      target_value: 75.0,
      predefined_metric_specification: {
        predefined_metric_type: 'DynamoDBWriteCapacityUtilization'
      },
      scale_out_cooldown: 1,
      scale_in_cooldown: 1
    }
  )
)

```

