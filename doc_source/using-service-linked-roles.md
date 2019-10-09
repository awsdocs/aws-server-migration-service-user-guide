# Service\-Linked Roles for AWS SMS<a name="using-service-linked-roles"></a>

AWS SMS uses a service\-linked role for the permissions that it requires to call other AWS services on your behalf\. For more information, see [Using Service\-Linked Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html) in the *IAM User Guide*\.

Before the introduction of a service\-linked role for AWS SMS, you were required to create two IAM roles to grant AWS SMS the permissions that it needs\. These roles are no longer required to use AWS SMS\. However, they are documented here for completeness\. For more information, see [Legacy IAM Roles for AWS SMS](sms-legacy-iam-roles.md)\.

## Permissions Granted by the Service\-Linked Role<a name="service-linked-role-permissions"></a>

AWS SMS uses the service\-linked role named **AWSServiceRoleForSMS** to enable AWS SMS to manage your replication jobs\.

**AWSServiceRoleForSMS** trusts the `sms.amazonaws.com` service principal to assume the role\.

The role permissions policy allows AWS SMS to complete actions on resources as follows:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "cloudformation:CreateChangeSet",
                "cloudformation:CreateStack",
                "cloudformation:DeleteStack",
                "cloudformation:ExecuteChangeSet"
            ],
            "Resource": "arn:aws:cloudformation:*:*:stack/sms-app-*/*",
            "Effect": "Allow",
            "Condition": {
                "ForAllValues:StringLikeIfExists": {
                    "cloudformation:ResourceTypes": [
                        "AWS::EC2::*"
                    ]
                }
            }
        },
        {
            "Action": [
                "cloudformation:DeleteChangeSet",
                "cloudformation:DescribeChangeSet",
                "cloudformation:DescribeStackEvents",
                "cloudformation:DescribeStackResources",
                "cloudformation:GetTemplate"
            ],
            "Resource": "arn:aws:cloudformation:*:*:stack/sms-app-*/*",
            "Effect": "Allow"
        },
        {
            "Action": [
                "cloudformation:DescribeStacks",
                "cloudformation:ValidateTemplate",
                "cloudformation:DescribeStackResource",
                "s3:ListAllMyBuckets"
            ],
            "Resource": "*",
            "Effect": "Allow"
        },
        {
            "Action": [
                "s3:CreateBucket",
                "s3:DeleteBucket",
                "s3:DeleteObject",
                "s3:GetBucketAcl",
                "s3:GetBucketLocation",
                "s3:GetObject",
                "s3:ListBucket",
                "s3:PutObject",
                "s3:PutObjectAcl",
                "s3:PutLifecycleConfiguration"
            ],
            "Resource": "arn:aws:s3:::sms-app-*",
            "Effect": "Allow"
        },
        {
            "Action": [
                "sms:CreateReplicationJob",
                "sms:DeleteReplicationJob",
                "sms:GetReplicationJobs",
                "sms:GetReplicationRuns",
                "sms:GetServers",
                "sms:ImportServerCatalog",
                "sms:StartOnDemandReplicationRun",
                "sms:UpdateReplicationJob"
            ],
            "Resource": "*",
            "Effect": "Allow"
        },
        {
            "Action": [
                "ec2:ModifySnapshotAttribute",
                "ec2:CopySnapshot",
                "ec2:CopyImage",
                "ec2:Describe*",
                "ec2:DeleteSnapshot",
                "ec2:DeregisterImage",
                "ec2:RunInstances"
            ],
            "Resource": "*",
            "Effect": "Allow"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:CreateTags",
                "ec2:DeleteTags"
            ],
            "Resource": "arn:aws:ec2:*:*:instance/*"
        },
        {
            "Action": "iam:GetRole",
            "Resource": "*",
            "Effect": "Allow"
        },
        {
            "Action": "iam:PassRole",
            "Resource": "*",
            "Effect": "Allow",
            "Condition": {
                "StringLike": {
                    "iam:AssociatedResourceArn": "arn:aws:cloudformation:*:*:stack/sms-app-*/*"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:ModifyInstanceAttribute",
                "ec2:StopInstances",
                "ec2:StartInstances",
                "ec2:TerminateInstances"
            ],
            "Resource": "*",
            "Condition": {
                "ForAllValues:StringLike": {
                    "ec2:ResourceTag/aws:cloudformation:stack-id": "arn:aws:cloudformation:*:*:stack/sms-app-*/*"
                }
            }
        }
    ]
}
```

## Create the Service\-Linked Role<a name="create-service-linked-role"></a>

You don't need to manually create the **AWSServiceRoleSMS** role\. AWS SMS creates this role for you when you select the **Allow automatic role creation** option when creating or updating a replication job, application, or launch configuration using the AWS Management Console\.

For AWS SMS to create a service\-linked role on your behalf, you must have the required permissions\. For more information, see [Service\-Linked Role Permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html#service-linked-role-permissions) in the *IAM User Guide*\.

If you do not use the console, you can create this service\-linked role manually\. For example, use the following AWS CLI [create\-service\-linked\-role](https://docs.aws.amazon.com/cli/latest/reference/iam/create-service-linked-role.html) command to create **AWSServiceRoleSMS**\.

```
aws iam create-service-linked-role --aws-service-name sms.amazonaws.com
```

## Edit the Service\-Linked Role<a name="edit-service-linked-role"></a>

You can edit the description of **AWSServiceRoleForSMS** using IAM\. For more information, see [Editing a Service\-Linked Role](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html#edit-service-linked-role) in the *IAM User Guide*\.

## Delete the Service\-Linked Role<a name="delete-service-linked-role"></a>

If you no longer need to use AWS SMS, we recommend that you delete the **AWSServiceRoleForSMS** role\. The service\-linked role can only be deleted in the following conditions:
+ The service\-linked role is not being used by an active replication job
+ The service\-linked role is not being used by an application that has an associated active replication job
+ The service\-linked role is not being used by an application that has an associated AWS CloudFormation stack

You can use the IAM console, the IAM CLI, or the IAM API to delete service\-linked roles\. For more information, see [Deleting a Service\-Linked Role](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html#delete-service-linked-role) in the *IAM User Guide*\.

After you delete the **AWSServiceRoleForSMS** role, AWS SMS creates the role again if you start a replication job\.