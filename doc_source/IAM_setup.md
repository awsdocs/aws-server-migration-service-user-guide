# Configure Your AWS Account Permissions<a name="IAM_setup"></a>

The following prerequisites apply to either platform supported by AWS SMS\.

If your IAM user account, group, or role is assigned administrator permissions, then you have access to AWS SMS\. To call the SMS API with the credentials of an IAM user that does not have administrative access to your AWS account, create a custom inline policy defined by the following JSON code and apply it to the IAM user:

```
{
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "sms:*"
                ],
                "Resource": "*"
            }
        ]
    }
```

**Note**  
If you are using multiple connectors, we recommend that you create a unique IAM role for each connector to avoid having a single point of failure\.

**To create an IAM user in your AWS account**

1. Create a new IAM user for your connector to communicate with AWS\. Save the generated access key and secret key for use during the initial connector setup\. For information about managing IAM users and permissions, see [Creating an IAM User in Your AWS Account](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html)\.

1. Attach the managed IAM policy `ServerMigrationConnector` to the IAM user\. For more information, see [Managed Policies and Inline Policies](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html)\.

1. Use one of the following procedures to create an IAM role that grants permissions to AWS SMS to place migrated resources into your Amazon EC2 account\. In AWS Regions that make an IAM role template available, **option 1** works\. If you find that no template for **AWS Server Migration Service** exists in your AWS Region, proceed to **option 2**\.

**To create an IAM role with a template \(option 1\)**

1. Open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the navigation pane, choose **Roles**, **Create role**\.

1. Under **Choose the service that will use this role**, choose **SMS**, **Next: Permissions**\. 

1. Under **Attached permissions policies**, confirm that the policy **ServerMigrationServiceRole** is visible and choose **Next: Review**\.

1. Under **Review**, for **Role name**, type **sms** \(recommended\)\. \(Optional\) You can apply a different name, but you must then specify the role name explicitly each time that you create a replication job\.

1. Choose **Create role**\. You should now see the **sms** role in the list of available roles\.

Use the following option in AWS Regions that do not make an IAM role template available\. This option also works as a manual alternative to **option 1** in all Regions\.

**To create an IAM role manually \(option 2\)**

1. Create a local file named `trust-policy.json` with the following content:

   ```
   {
           "Version": "2012-10-17",
           "Statement": [
               {
                   "Sid": "",
                   "Effect": "Allow",
                   "Principal": {
                       "Service": "sms.amazonaws.com"
                   },
                   "Action": "sts:AssumeRole",
                   "Condition": {
                       "StringEquals": {
                           "sts:ExternalId": "sms"
                       }
                   }
               }
           ]
       }
   ```

1. Create a local file named `role-policy.json` with the following content:

   ```
   {
           "Version": "2012-10-17",
           "Statement": [
               {
                   "Effect": "Allow",
                   "Action": [
                       "ec2:ModifySnapshotAttribute",
                       "ec2:CopySnapshot",
                       "ec2:CopyImage",
                       "ec2:Describe*",
                       "ec2:DeleteSnapshot",
                       "ec2:DeregisterImage",
                       "ec2:CreateTags",
                       "ec2:DeleteTags"
                   ],
                   "Resource": "*"
               }
           ]
       }
   ```

1. At a command prompt, go to the directory where you stored the two JSON policy files, and run the following commands to create the AWS SMS service role:

   ```
   aws iam create-role --role-name sms --assume-role-policy-document file://trust-policy.json
   aws iam put-role-policy --role-name sms --policy-name sms --policy-document file://role-policy.json
   ```

**Note**  
Your AWS CLI user must have permissions on IAM\. You can grant these by attaching the `IAMFullAccess` managed policy to your AWS CLI user\. For information about managing IAM users and permissions, see [Creating an IAM User in Your AWS Account](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html)\. 