# Run code before terminating an EC2 Auto Scaling instance

CloudFormation template related to [this blog]() published on the AWS [Infrastructure & Automation blog](https://aws.amazon.com/blogs/infrastructure-and-automation/)

## Overview
This is a sample solution using [Amazon EC2 Auto Scaling Lifecycle Hooks](https://docs.aws.amazon.com/autoscaling/ec2/userguide/lifecycle-hooks.html) to perform any desired actions before terminating the instance within the Auto Scaling group. [lifecycle hook](https://docs.aws.amazon.com/autoscaling/ec2/userguide/lifecycle-hooks.html#adding-lifecycle-hooks) puts the instance in `Terminating:Wait` status. The `Terminating:Wait` status will be monitored by an Amazon CloudWatch event, which triggers an AWS Systems Manager automation document to perform the action you want.

## Deployment
### Prerequisites
- An Amazon EC2 Auto Scaling group.
- A [string parameter](https://docs.aws.amazon.com/systems-manager/latest/userguide/param-create-cli.html#param-create-cli-string-stringlist) for the domain user. The user must have permission to remove the computer from the domain. I refer to this parameter as DomainUserName.
- A [secure string parameter](https://docs.aws.amazon.com/systems-manager/latest/userguide/param-create-cli.html#param-create-cli-securestring) that contains the DomainUserName password. I refer to this parameter as DomainPassword.

### Walkthrough

[RunSSMAutomationBeforeTermination.json CFN](https://github.com/aws-samples/aws-systemsmanagerautomation-with-asglifecyclehooks/blob/master/RunSSMAutomationBeforeTermination.json) will go through the following steps:
1.	Add a [lifecycle hook](https://docs.aws.amazon.com/autoscaling/ec2/userguide/lifecycle-hooks.html#adding-lifecycle-hooks).
2.	Create a [Systems Manager automation document](https://docs.aws.amazon.com/systems-manager/latest/userguide/automation-document-builder.html). The automation document goes through the following steps.
    - Run a Windows PowerShell script to remove the computer from the domain.
    - Create an AMI of the EC2 instance.
    - Terminate the instance.
3.	Create AWS Identity and Access Management (IAM) [policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html) and a [role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html) to delegate permissions to the Systems Manager automation document.
4.	Create IAM policies and a role to delegate permissions to Amazon CloudWatch Events, which invokes the Systems Manager automation document.
5.	Create a [CloudWatch Events rule](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/Create-CloudWatch-Events-Rule.html).
6.	Add a [Systems Manager automation document as a CloudWatch Event target](https://docs.aws.amazon.com/systems-manager/latest/userguide/automation-cwe-target.html).

For more details about launching a stack, refer to [Creating a Stack on the AWS CloudFormation Console](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html).

### Template Parameters
The stack template includes the following parameters:

| Parameter | Required | Description |
| --- | --- | --- |
| AutoScalingGN | Yes | Enter the name of the auto scaling group to monitor and add the [lifecycle hook](https://docs.aws.amazon.com/autoscaling/ec2/userguide/lifecycle-hooks.html#adding-lifecycle-hooks). |
| LifecycleHookName | Yes | The name of the lifecycle hook. |
| HeartbeatTimeout | Yes | Set the heartbeat timeout for the lifecycle hook when you create the lifecycle hook. |
| CloudWatchEventName | Yes | The name of the CloudWatch Event Rule. |
| CloudWatchEventDescription | No | The description of the CloudWatch Event rule. |
| ExistingCloudWatchEventRole | No | The Role ARN to be used by CloudWatch event to trigger the AWS Systems Manager Automation execution.If not specified, the template will create a rule with minimum permissions as describe in the blog. |
| ExistingAutomationAssumeRole | No | The ARN of the role that allows AWS Systems Manager Automation execution to perform the actions in the document. If not specified, the template will create a rule with minimum permissions as describe in the blog. |
| DomainUserName | Yes | The name of the string Parameter for the Domainuser. The user would need to have enough permissions to remove the computer from the domain. |
| DomainPassword | Yes | The name of the Secure Parameter that have the password of DomainUserName. |

## License

This library is licensed under the MIT-0 License. See the LICENSE file.