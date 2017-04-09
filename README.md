# AWS Application Load Balancer virtual host routing for CloudFormation

`cfn-alb-host` is a [Lambda-backed custom resource][cfn-res] that makes it
dead simple to use a single load balancer with different backends per hostname.

## Usage

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Resources:
  # have a target group and ALB listener defined somewhere...
  MyVhost:
    Type: Custom::AlbVhost
    Properties:
      ServiceToken: !ImportValue AlbVhost
      HostHeader: mysubdomain.example.com
      TargetGroupArn: !Ref TargetGroup
      ListenerArn: !Ref Listener

  # if you are associating an EC2 Container Service (ECS) service
  # with the target group, you will want to add a dependency on
  # the vhost - ECS will complain about an unassociated load balancer
  # otherwise.
  Service:
    Type: AWS::ECS::Service
    DependsOn: [MyVhost]
    Properties:
      LoadBalancers:
        - ContainerName: # whatevs
          ContainerPort: # whatevs
          TargetGroupArn: !Ref TargetGroup
```

Note that neither CloudFormation, Lambda nor Load Balancers are global resources,
so you will have to deploy the helper stack into each region that you wish to 
use this in.

[cfn-res]: http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-custom-resources-lambda.html
