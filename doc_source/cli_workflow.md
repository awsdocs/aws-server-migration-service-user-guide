# Replicating VMs Using the CLI<a name="cli_workflow"></a>

This topic provides a CLI\-based example of the workflow involved in using AWS SMS to inventory and migrate your on\-premises servers to Amazon EC2\.

**To replicate a server using the CLI**

1. Install the Server Migration Connector as described in [Getting Started with AWS Server Migration Service](SMS_setup.md), including the configuration of an IAM service role and permissions\.

1. Use the get\-connectors command to obtain a list of connectors that are registered to you\.

   ```
   aws sms get-connectors --region us-east-1
   ```

1. After a connector has been installed and registered through the console, use the import\-server\-catalog command to create an inventory of your servers\. This process can take up to a minute\.

   ```
   aws sms import-server-catalog --region us-east-1
   ```
**Note**  
There are currently no CLI commands for installing or registering a connector\.

1. Use the get\-servers command to display a list of servers available for import to Amazon EC2\.

   ```
   aws sms get-servers --region us-east-1
   ```

   The output should be similar to the following:

   ```
   {
       "serverList": [
           {
               "serverId": "s-12345678", 
               "serverType": "VIRTUAL_MACHINE", 
               "vmServer": {
                   "vmManagerName": "vcenter.yourcompany.com", 
                   "vmServerAddress": {
                       "vmManagerId": "your-vcenter-instance-uuid", 
                       "vmId": "vm-123"
                   }, 
                   "vmName": "your-linux-vm", 
                   "vmPath": "/Datacenters/DC1/vm/VM Folder Path/your-linux-vm", 
                   "vmManagerType": "vSphere"
               }
           }, 
           {
               "replicationJobTerminated": false, 
               "serverId": "s-23456789", 
               "serverType": "VIRTUAL_MACHINE", 
               "replicationJobId": "sms-job-12345678", 
               "vmServer": {
                   "vmManagerName": "vcenter.yourcompany.com", 
                   "vmServerAddress": {
                       "vmManagerId": "your-vcenter-instance-uuid", 
                       "vmId": "vm-234"
                   }, 
                   "vmName": "Your Windows VM", 
                   "vmPath": "/Datacenters/DC1/vm/VM Folder Path/Your Windows VM", 
                   "vmManagerType": "vSphere"
               }
           }
       ]
   }
   ```

   If you have not yet imported a server catalog, you see output similar to the following:

   ```
   {
       "lastModifiedOn": 1477006131.856, 
       "serverCatalogStatus": "NOT IMPORTED", 
       "serverList": []
   }
   ```

   A catalog status of DELETED or EXPIRED also shows that no servers exist in the catalog\.

1. Select a server to replicate, note the server ID, and use that as a parameter in the create\-replication\-job command\.

   ```
   aws sms create-replication-job --region us-east-1 --server-id s-12345678 --frequency 12 --seed-replication-time 2016-10-24T15:30:00-07:00
   ```

   After the replication job is set up, it starts replicating automatically at the time specified with the `--seed-replication-time` parameter, expressed in seconds of the Unix epoch or according to ISO 8601\. For more information, see [Specifying Parameter Values for the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/cli-using-param.html)\. Thereafter, the replication repeats with an interval specified by the `--frequency` parameter, expressed in hours\. 

1. You can view details of all running replication jobs using the get\-replication\-jobs command\. If this command is used without parameters, it returns a list of all your replication jobs\.

   For example, the get\-replication\-jobs command returns information similar to the following:

   ```
   {
       "replicationJobList": [
           {
               "vmServer": {
                   "vmManagerName": "vcenter.yourcompany.com", 
                   "vmServerAddress": {
                       "vmManagerId": "your-vcenter-instance-uuid", 
                       "vmId": "vm-1234"
                   }, 
                   "vmName": "VM name in vCenter", 
                   "vmPath": "/Datacenters/DC1/vm/VM Folder Path/VM name in vCenter"
               }, 
               "replicationRunList": [
                   {
                       "scheduledStartTime": 1487007010.0, 
                       "state": "Deleted", 
                       "type": "Automatic", 
                       "statusMessage": "Uploading", 
                       "replicationRunId": "sms-run-12345678"
                   }
               ], 
               "replicationJobId": "sms-job-98765432", 
               "state": "Deleted", 
               "frequency": 12, 
               "seedReplicationTime": 1477007049.0, 
               "roleName": "sms"
           }, 
           {
               "vmServer": {
                   "vmManagerName": "vcenter.yourcompany.com", 
                   "vmServerAddress": {
                       "vmManagerId": "your-vcenter-instance-uuid", 
                       "vmId": "vm-2345"
                   }, 
                   "vmName": "win2k12", 
                   "vmPath": "/Datacenters/DC1/vm/VM Folder Path/win2k12"
               }, 
               "replicationRunList": [
                   {
                       "scheduledStartTime": 1477008789.0, 
                       "state": "Active", 
                       "type": "Automatic", 
                       "statusMessage": "Converting", 
                       "replicationRunId": "sms-run-12345679"
                   }
               ], 
               "replicationJobId": "sms-job-23456789", 
               "state": "Active", 
               "frequency": 24, 
               "seedReplicationTime": 1477008789.0, 
               "roleName": "sms"
           }
       ]
   }
   ```

   This command returns a paginated response, with 50 items per page as the default\. You may also specify a custom page length with the `--max-items` parameter, which takes an integer value denoting the number of items to return on one page\.

1. You can also use the get\-replication\-runs command to retrieve details on all replication runs for a specific replication job\. To do this, pass in a replication job ID to the command as follows:

   ```
   aws sms get-replication-runs --replication-job-id sms-job-12345678 --region us-east-1
   ```

   This command returns a list of all replication runs for the specified replication job, as well as details for that replication job, similar to the following:

   ```
   {
       "replicationRunList": [
           {
               "scheduledStartTime": 1477310423.0,
               "state": "Active",
               "type": "Automatic",
               "statusMessage": "Converting",
               "replicationRunId": "sms-run-23456789"
           },
           {
               "amiId": "ami-abcdefab",
               "state": "Completed",
               "completedTime": 1477227683.652,
               "scheduledStartTime": 1477224023.0,
               "replicationRunId": "sms-run-34567890",
               "type": "Automatic",
               "statusMessage": "Completed"
           },
           {
               "amiId": "ami-efababcd",
               "state": "Completed",
               "completedTime": 1477144823.486,
               "scheduledStartTime": 1477137623.0,
               "replicationRunId": "sms-run-45678903",
               "type": "Automatic",
               "statusMessage": "Completed"
           }
       ]
   }
   ```

   As with the plain get\-replication\-jobs call, this call returns paginated results\.

1. To change any of the parameters of a replication job after you have created it, use the update\-replication\-job command, by providing the replication job ID and any parameters to change\.

   ```
   aws sms update-replication-job --region us-east-1 --replication-job-id sms-job-12345678 --frequency 24 --next-replication-run-start-time 2016-10-24T15:30:00-07:00
   ```

1. In addition to your scheduled replication runs, you may also start up to two on\-demand replication runs per 24\-hour period\. To do this, use the start\-on\-demand\-replication\-run command, which starts a replication run immediately\. If another replication run is currently active, an on\-demand replication run cannot be started\. 

   ```
   aws sms start-on-demand-replication-run --replication-job-id sms-job-12345678 --region us-east-1
   ```

   If a scheduled replication run is expected to start while an on\-demand replication run is ongoing, then the scheduled run is skipped and rescheduled for the next interval\.

1. After you are finished replicating a server, you may stop the replication job using the delete\-replication\-job command\. This stops the replication job and cleans up any artifacts created by the service \(for example, the job's S3 bucket\)\. This does not delete any AMIs created by runs of the stopped job\.

   ```
   aws sms delete-replication-job --region us-east-1 --replication-job-id sms-job-12345678
   ```

1. When you no longer need to maintain your catalog of servers, use the delete\-server\-catalog command to clear the catalog of servers maintained by the service\.

   ```
   aws sms delete-server-catalog --region us-east-1
   ```

1. When you are done using a connector, use the disassociate\-connector command to deregister the connector from AWS SMS\. Call this command only after all replications using that connector are complete\.

   ```
   aws sms disassociate-connector --region us-east-1 --connector-id c-12345678901234567
   ```