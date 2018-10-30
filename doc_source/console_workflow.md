# Replicating VMs Using the AWS SMS Console<a name="console_workflow"></a>

Use the AWS SMS console to import your server catalog and migrate your on\-premises servers to Amazon EC2\. You can perform the following tasks:
+ [Replicate a server using the console](#configure_replication)
+ [Monitor and modify server replication jobs](#monitor_replication)
+ [Shut down replication](#delete_replication) <a name="configure_replication"></a>

**To replicate a server using the console**

1. Install the Server Migration Connector as described in [Getting Started with AWS Server Migration Service](SMS_setup.md), including the configuration of an IAM service role and permissions\.

1. In a web browser, open the [SMS homepage](https://console.aws.amazon.com/servermigration/home)\.
**Tip**  
If this link takes you to the AWS SMS setup page, trim the "gettingStarted" off of the end of the URL and press return\.

1. In the navigation menu, choose **Connectors**\. Verify that the connector you deployed in your VMware environment is shown with a status of healthy\.

1. If you have not yet imported a catalog, choose **Servers**, **Import server catalog**\. To reflect new servers added in your VMware environment after your previous import operation, choose **Re\-import server catalog**\. This process can take up to a minute\.

1. Select a server to replicate and choose **Create replication job**\.

1. On the **Configure server\-specific settings** page, in the **License type** column, select the license type for AMIs to be created from the replication job\. Linux servers can only use Bring Your Own License \(BYOL\)\. Windows servers can use either an AWS\-provided license or BYOL\. You can also choose **Auto** to allow AWS SMS to select the appropriate license\. Choose **Next**\.

1. On the **Configure replication job settings** page, the following settings are available:
   + For **Replication job type**, choose a value\. The **replicate server every *interval*** option creates a repeating replication process that creates new AMIs at the interval you provide from the menu\. The **One\-time migration** option triggers a single replication of your server without scheduling repeating replications\.
   + For **Start replication run**, configure your replication run to start either immediately or at a later date and time up to 30 days in the future\. The date and time settings refer to your browser’s local time\. 
   + For **IAM service role**, provide \(if necessary\) the IAM service role that you previously created\.
   + \(Optional\) For **Description**, provide a description of the replication run\.
   + For **Enable automatic AMI deletion**, configure AWS SMS to delete older replication AMIs in excess of a number that you provide in the field\.
   + For **Enable notifications**, choose a value\. If you choose **Yes**, you can configure Amazon Simple Notification Service \(Amazon SNS\) to notify a list of recipients when the replication job has completed, failed, or been deleted\. For more information, see [What is Amazon Simple Notification Service?](http://docs.aws.amazon.com/sns/latest/dg/)\.
   + For **Pause replication job on consecutive failures**, choose a value\. The default is set to **Yes**\. This will ensure that when the replication runs for the job encounter consecutive failures, the job will be moved to the **PausedOnFailure** state and not marked **Failed** immediately\.

   Choose **Next**\.

1. On the **Review** page, review your settings\. If the settings are correct, choose **Create**\. To change the settings, choose **Previous**\. After a replication job is set up, replication starts automatically at the specified time and interval\.

In addition to your scheduled replication runs, you may also start up to two on\-demand replication runs per 24\-hour period\. On the **Replication jobs** page, select a job and choose **Actions**, **Start replication run**\. This starts a replication run that does not affect your scheduled replication runs, except in the case that the on\-demand run is still ongoing at the time of your scheduled run\. In this case, the scheduled run is skipped and rescheduled at the next interval\. The same thing happens if a scheduled run is due while a previous scheduled run is still in progress\.<a name="monitor_replication"></a>

**To resume replication jobs that are paused**

1. Before attempting to resume a job that is in **PausedOnFailure** state, kindly refer to the [SMS Troubleshooting guide](troubleshoot-sms.md) on how to diagnose the replication run failures when a job is in the **PausedOnFailure** state\. It is advisable to address the root cause of the failure before resuming the replication job\.

1. In the AWS SMS console, choose **Replication jobs**\. You can view all replication jobs by scrolling through the table\. In the search bar, you can filter the table contents on specific values\. Filter the jobs by the **PausedOnFailure** state to identify all the paused jobs\.

1. To resume a paused job, select the job on the **Replication jobs** page and choose **Actions**, **Resume replication job**\.

**To monitor and modify server replication jobs**

1. In the AWS SMS console, choose **Replication jobs**\. You can view all replication jobs by scrolling through the table\. In the search bar, you can filter the table contents on specific values\. 

1. Select a single replication job to view details about it in the lower pane\. The **Job details** tab displays information about the current replication run, including the ID of the latest AMI created by the replication job\. The **Run history** tab shows details about all of the replication runs for the selected replication job\. 

1. To change any job parameters, select a job on the **Replication jobs** page and choose **Actions**, **Edit replication job**\. After entering new information in the **Edit configuration job** form, choose **Save** to commit your changes\. 
**Note**  
You may need to refresh the page for the changes to become visible\.<a name="delete_replication"></a>

**To shut down replication**

1. After you have finished replicating a server, you can delete the replication job\. Choose **Replication jobs**, select the desired job, choose **Actions**, and then choose **Delete replication jobs**\. In the confirmation window, choose **Delete**\. This stops the replication job and cleans up any artifacts created by the service \(for example, the job's S3 bucket\)\. This does not delete any AMIs created by runs of the stopped job\. 
**Note**  
You may need to refresh the page for the changes to become visible\.

1. To clear your server catalog after you no longer need it, choose **Servers**, **Clear server catalog**\. The list of servers is removed from AWS SMS and your display\. 

1. When you are done using a connector and no longer need it for any replication jobs, you can disassociate it\. Choose **Connectors** and select the connector to disassociate\. Choose **Disassociate** at the top\-right corner of its information section and choose **Disassociate** again in the confirmation window\. This action deregisters the connector from AWS SMS\. 
