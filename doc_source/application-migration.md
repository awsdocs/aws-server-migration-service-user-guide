# Migrate Applications Using AWS SMS<a name="application-migration"></a>

AWS Server Migration Service supports the automated migration of multi\-server application stacks from your on\-premises data center to Amazon EC2\. Where server migration is accomplished by replicating a single server as an Amazon Machine Image \(AMI\), application migration replicates all of the servers in an application as AMIs and generates an AWS CloudFormation template to launch them in a coordinated fashion\.

Applications can be further subdivided into groups that allow you to launch tiers of servers in a defined order\. The following diagram provides a sample case of a database\-backed web application:

![\[Tiered launching of an application using groups.\]](http://docs.aws.amazon.com/server-migration-service/latest/userguide/images/app_migration.png)

In this example, the application is divided into four groups, each with three servers\. The AWS CloudFormation template starts the servers in the following order: databases, file servers, web servers, and application servers\.

After your servers are organized into applications and launch groups, you can specify a replication frequency, provide configuration scripts, and configure a target VPC in which to launch them\. When you launch an application, AWS SMS configures it based on the generated template\.

Application migration relies on the procedures for discovering on\-premises resources described in [Install the Server Migration Connector](SMS_setup.md)\. After you have imported a server catalog into AWS SMS using the Server Migration Connector, you can configure settings for applications, replication, and launch, as well as monitor migration status, in the **Applications** section of the AWS SMS console\. You can also perform these tasks using the resources for AWS SMS in the AWS SMS API, AWS CLI, or AWS SDKs\.

You can replicate your on\-premises servers to AWS for up to 90 days per server\. Usage time is calculated from the time a server replication begins until you terminate the replication job\. After 90 days, your replication job is automatically terminated\. You can request an extension from AWS Support\.

**Note**  
Application migration from Microsoft Azure environments is supported, but the Server Migration Connector for Azure does not currently guarantee the closeness of the server snapshots in the application\. 

## Using Application Migration<a name="using-application-migration"></a>

This section provides step\-by\-step procedures for creating, configuring, replicating, and launching applications\.

**To create an application**

1. Open the AWS SMS console at [https://console\.aws\.amazon\.com/servermigration/](https://console.aws.amazon.com/servermigration/)\.

1. Choose **Applications**\. On the **Applications** page, you can view your existing applications \(if any\)\. 

1. Choose **Create new application**\. 

1. On the **Create new application** page, under **Application settings**, supply the following information and then choose **Next**:
   + **Application name** — Specify a name for the application\.
   + **Application description** — Optionally, specify a description for the application\.
   + **Role name** — Select **Allow automation role creation** to have AWS SMS create a service\-linked role on your behalf\. For more information, see [Service\-Linked Roles for AWS SMS](using-service-linked-roles.md)\. Select **Use my own role** to specify an existing IAM role to use\.

1. Under **Select servers**, select the available servers to include in the application\. In the search bar, you can filter the table contents on specific values\. Choose **Next: Add servers to groups**\.
**Note**  
Ungrouped servers are added to a default group\.

1. Under **Add servers to groups**, you can create groups, delete groups, add selected servers from your application to groups, and remove servers from groups\. 

   Complete the following steps to add one or more servers to a new group:

   1. Select the servers to be added to the new group\.

   1. Choose **Add servers to group**\.

   1. Under **Add servers to group**, choose **Add to new group** and provide a name for the group\.

   1. Choose **Add**\. The list of servers now displays the associated group for each server that you selected\.

1. After creating one or more groups, you can delete a group by completing the following steps:

   1. Choose **Delete group**\.

   1. For **Group to delete**, choose a group\.

   1. Choose **Delete**\.

   Deleting a group has no effect on servers that belong to it\.

1. Under **Add tags**, tag your applications with key/value pairs that propagate to all of the servers created when the application is launched\. Choose **Next**\. 

1. Under **Review**, you can review and edit your application and group settings\. When you are satisfied that the settings are correct, choose **Create**\. From the status page, you can proceed directly to **Configure replication settings**\.

**To configure replication settings for an application**

1. Open the AWS SMS console at [https://console\.aws\.amazon\.com/servermigration/](https://console.aws.amazon.com/servermigration/)\.

1. Choose **Applications**\. On the **Applications** page, you can view the available applications\.

1. Select the name of the application to configure\.

1. Choose **Actions**, **Configure replication settings**\.

1. On the **Replication settings** page, provide the following information and then choose **Next**:
   + **Replication job type** — Specify the replication period \(1\-24 hours\) or choose **One\-time replication**\.
   + **Start replication run** — Choose to start a replication run immediately, or choose **At a later time and date** and enter the information\. 
   + **Enable automatic AMI deletion** — Choose **Yes** or **No**\.

1. The **Server\-specific settings** page displays the application servers and their group memberships\. You can edit the following server settings individually or together:
   + **License type** — Choose **Auto**, **AWS**, or **BYOL**\.
   + **Quiesce** — Before taking a snapshot of the VM, halt data input/output and store the system memory state \(for VMware\)\.

1. Choose **Next**\.

1. Review the replication settings and choose **Save**\. From the status page, you can proceed directly to **Configure launch settings**\.

**To configure launch settings for an application**

1. Open the AWS SMS console at [https://console\.aws\.amazon\.com/servermigration/](https://console.aws.amazon.com/servermigration/)\.

1. Choose **Applications**\. On the **Applications** page, you can view the available applications\.

1. Select the name of the application to configure\.

1. Choose **Actions**, **Configure launch settings**\.

1. On the **Configure launch settings** page, for **IAM CloudFormation role**, specify a non\-default value\. Under **Specify launch order**, configure a launch order for your groups\. Choose **Next**\.

1. Under **Configure launch settings** for the application, you can edit the following server settings individually or multiply:
   + **Logical ID** — AWS CloudFormation resource ID\. This parameter is used as the logical ID of the CloudFormation template that AWS SMS generates for the application\. A value is created automatically when you use the console, but you must supply it manually when using the API, CLI, or SDKs\. For more information, see [Resources](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/resources-section-structure.html) in the *AWS CloudFormation User Guide*\.
   + **Instance type** — Specifies the EC2 instance type on which to launch the server\. This field is required\.
   + **Key pair** — Specifies the SSH key pair that gives access to the server\. This field is required\.
**Note**  
If the logged\-in user does not have IAM permissions to list key pairs, this list will be empty\.
   + **Configuration script** — A script to run configuration commands at startup of EC2 instances launched as part of an application\. 

   Choose **Next**\.

1. Under **Configure target network and security** settings for the application, you can edit the following server settings individually or multiply\. Network settings require prior setup as described in [RunInstances](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_RunInstances.html)\.
**Note**  
If the logged\-in user does not have IAM permissions to list VPCs, subnets, security groups, these lists will be empty\.
   + **VPC** — VPC in which to deploy the application\. This field is required\.
   + **Subnet** — Subnet in which to deploy the application\. This field is required\.
   + **Security Group** — Security group membership for the application\. This field is required\.
   + **Publicly Accessible** — Whether the application should be accessible from the internet\.

   Choose **Next**\.

1. Review the launch configuration settings and choose **Save**\.

**To start replicating an application**

1. Open the AWS SMS console at [https://console\.aws\.amazon\.com/servermigration/](https://console.aws.amazon.com/servermigration/)\.

1. Choose **Applications**\. On the **Applications** page, you can view the available applications\.

1. Choose the name of the application to replicate\.

1. On the application details page, choose **Actions**, **Start replication**\.

1. In the **Start replication** window, choose **Start**\. Replication can take anywhere from 30 minutes to several days depending on the disk size\. On the application details page, you can observe the status of the replication in the **Replication status** field\. If the replication fails, you may be able to find the reason in the **Replication status message** field\.

**To launch an application**

1. Open the AWS SMS console at [https://console\.aws\.amazon\.com/servermigration/](https://console.aws.amazon.com/servermigration/)\.

1. Choose **Applications**\. On the **Applications** page, you can view the available applications\.

1. Choose the name of the application to launch\.

1. On the application details page, choose **Actions**, **Launch application**\. A replication job must complete before you perform this action\.

1. In the **Launch application** window, choose **Launch**\. On the application details page, you can observe the status of the launch in the **Launch status** field\. If the launch fails, you may be able to find the reason in the **Launch status message** field\.

**To generate an AWS CloudFormation template for the application**

Use the following procedure if you want to examine the AWS CloudFormation template that is auto\-generated when you launch the application\.

1. Open the AWS SMS console at [https://console\.aws\.amazon\.com/servermigration/](https://console.aws.amazon.com/servermigration/)\.

1. Choose **Applications**\. On the **Applications** page, you can view the available applications\.

1. Choose the name of the application for which to create a template\.

1. On the application details page, choose **Actions**, **Generate template**\. A replication job must complete before you perform this action\.

1. In the **Generate template** window, choose **Generate**\.

## Importing Applications from Migration Hub<a name="migration-hub"></a>

Application Migration supports the import and migration of applications discovered by AWS Migration Hub\.

**To import applications from Migration Hub**

1. To enable application catalog import, complete the [AWS Server Migration Service \(SMS\)](https://docs.aws.amazon.com/migrationhub/latest/ug/new-customer-setup.html#sms-managed) instructions in the Migration Hub user guide\.
**Note**  
Taking this action exports the SMS server catalog and makes it visible on Migration Hub\.

1. In the SMS console, on the **Applications** page, choose **Import applications**\.

1. In the **Import applications** window, you can optionally provide a value in the **Role name** field\. If no role name is specified, the default role name *sms* is used\. Choose **Import**\.
**Note**  
SMS imports application\-related servers from Migration Hub only if they exist in the SMS Server Catalog and are not part of an existing SMS application\. As a result, some applications may be only partially imported\. 

1. After import completes, the applications imported from Migration Hub appear in the **Applications** table\. Imported applications can be migrated but cannot be edited in SMS\. They can, however, be edited in Migration Hub\. After editing, re\-import\.
**Note**  
An application cannot be re\-imported if it is being actively replicated or launched by SMS\. If this conflict occurs, stop the replication or launch and re\-import\.