# Run code before terminating an EC2 Auto Scaling instance

CloudFormation template related to [this blog](https://aws.amazon.com/blogs/infrastructure-and-automation/run-code-before-terminating-an-ec2-auto-scaling-instance/) published on the AWS [Infrastructure & Automation blog](https://aws.amazon.com/blogs/infrastructure-and-automation/)

## Overview
This is a sample solution using [Amazon EC2 Auto Scaling Lifecycle Hooks](https://docs.aws.amazon.com/autoscaling/ec2/userguide/lifecycle-hooks.html) to perform any desired actions before terminating the instance within the Auto Scaling group. [lifecycle hook](https://docs.aws.amazon.com/autoscaling/ec2/userguide/lifecycle-hooks-overview.html) puts the instance in `Terminating:Wait` status. The `Terminating:Wait` status will be monitored by an Amazon CloudWatch event, which triggers an AWS Systems Manager automation document to perform the action you want.

## Deployment
### Prerequisites
- An Amazon EC2 Auto Scaling group.
- A [String parameter](https://docs.aws.amazon.com/systems-manager/latest/userguide/param-create-cli.html#param-create-cli-string) for the domain user. The user must have permission to remove the computer from the domain. I refer to this parameter as DomainUserName.
- A [SecureString parameter](https://docs.aws.amazon.com/systems-manager/latest/userguide/param-create-cli.html#param-create-cli-securestring) that contains the DomainUserName password. I refer to this parameter as DomainPassword.

### Walkthrough

The CloudFormation template [RunSSMAutomationBeforeTermination.json](/RunSSMAutomationBeforeTermination.json) will go through the following steps:
1.	Add a [lifecycle hook](https://docs.aws.amazon.com/autoscaling/ec2/userguide/adding-lifecycle-hooks.html).
2.	Create a [Systems Manager automation document](https://docs.aws.amazon.com/systems-manager/latest/userguide/automation-documents.html). The automation document goes through the following steps.
    - Run a Windows PowerShell script to remove the computer from the domain.
    - Create an AMI of the EC2 instance.
    - Execute AWS API [CompleteLifecycleAction](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_CompleteLifecycleAction.html) to terminate the instance.
3.	Create a [CloudWatch Events rule](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-create-rule.html).
4.	Add a [Systems Manager automation document as a CloudWatch Event target](https://docs.aws.amazon.com/systems-manager/latest/userguide/running-automations-event-bridge.html).
5.	(Optional) Create AWS Identity and Access Management (IAM) [policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html) and a [role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html) to delegate permissions to the Systems Manager automation document.
6.	(Optional) Create AWS Identity and Access Management (IAM) [policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html) and a [role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html) to delegate permissions to Amazon CloudWatch Events, which invokes the Systems Manager automation document.

For more details about launching a stack, refer to [Creating a Stack on the AWS CloudFormation Console](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html).

### Template Parameters
The stack template includes the following parameters:

| Parameter | Required | Description |
| --- | --- | --- |
| AutoScalingGN | Yes | Enter the name of the auto scaling group to monitor and add the [lifecycle hook](https://docs.aws.amazon.com/autoscaling/ec2/userguide/lifecycle-hooks.html). |
| DomainUserName | Yes | The name of the [String parameter](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html#parameter-type-string) for the DomainUser. The user would need to have enough permissions to remove the computer from the domain. |
| DomainPassword | Yes | The name of the [SecureString parameter](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html#parameter-type-securestring) that have the password of DomainUserName. |
| ExistingAutomationAssumeRole | No | The ARN of [AWS Systems Manager Automation assume role](https://docs.aws.amazon.com/systems-manager/latest/userguide/automation-setup.html#automation-setup-configure-role). If not specified, the template will create a role with minimum permissions as describe in the blog. |
| ExistingCloudWatchEventRole | No | The Role ARN to be used by CloudWatch event to trigger the AWS Systems Manager Automation execution.If not specified, the template will create a role with minimum permissions as describe in the blog. |

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.
