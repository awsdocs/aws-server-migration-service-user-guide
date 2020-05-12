# Troubleshooting AWS SMS<a name="troubleshoot-sms"></a>

The following information can help you troubleshoot issues with errors that you might encounter when using AWS SMS\. Before using these procedures, confirm that your SMS setup and the server you are trying to migrate meet the requirements in [Server Migration Service \(SMS\) Requirements](prereqs.md)\.

**Topics**
+ [Failure When Registering the Connector](#failure-registering-connector)
+ [Certificate Error When Uploading a VM to Amazon S3](#sms-cert-mismatch)
+ [Server Migration Connector Fails To Connect To AWS with Error "PKIX path building failed"](#cert-re-signing)
+ [This CA Root certificate is not trusted](#ca-root-certificate-not-trusted)
+ [Replication Run Fails During the Preparing Stage](#preparing-failure)
+ [Replicated AMI Doesn't Support Some Instance Types for Launch](#unavailable-instance-types)
+ [Failure to upload to Amazon S3](#failure-uploading-base-disks)
+ [Incremental Replication Delta Exceeds 1 TB](#delta-migrations)

## Failure When Registering the Connector<a name="failure-registering-connector"></a>

If you encounter an issue registering the connector, contact [sms\-service@amazon\.com](mailto:sms-service@amazon.com)\.

## Certificate Error When Uploading a VM to Amazon S3<a name="sms-cert-mismatch"></a>

The connector may fail to replicate your VM because the VM is on an ESXi host with an SSL certificate problem\. If this occurs, you see the following error message displayed in the **Latest run's status message** section: "ServerError: Failed to upload base disk\(s\) to S3\. Please try again\. If this problem persists, please contact AWS Support: vSphere certificate hostname mismatch: Certificate for <*somehost\.somedomain\.com*> doesn't match any of the subject alternative names: \[*localhost\.localdomain*\]\."

You can override this ESXi host certificate problem by completing the following tasks:

**Topics**
+ [Upgrade Your Connector](#upgrade)
+ [Re\-Register Your Connector](#reregister)

### Upgrade Your Connector<a name="upgrade"></a>

This section is for customers who are manually upgrading the connector\. If you have previously configured automatic upgrades, skip these steps and continue to [Re\-Register Your Connector](#reregister)\.

**To upgrade your connector**

1. Open the connector console\.

1. Log in to the connector\.

1. Choose **Upgrade**\.

1. Wait for the connector to finish upgrading to version 1\.0\.11\.13 or later\.

### Re\-Register Your Connector<a name="reregister"></a>

This section applies to all customers encountering the certificate mismatch problem\.

**To re\-register your connector**

1. Open the connector console\.

1. Log in to the connector\.

1. In the **General Health** section, check that the connector version is 1\.0\.11\.13 or later\.

1. Choose **Edit AWS Server Migration Service Settings**\.

1. On the **Setup** page, for **AWS Region**, select the desired region from the list\. For **AWS Credentials**, enter the IAM access key and secret key that you created in Step 2 of the [setup guide](SMS_setup.md)\. Choose **Next**\.

1. On the **vCenter Service Account** page, enter the vCenter hostname, user name, and password that you created in Step 3 of the [setup guide](SMS_setup.md)\.

1. Select the **Ignore hostname mismatch and expiration errors for vCenter and ESXi certificates** check box\. Choose **Next**\.

1. Complete registration and view the connector configuration dashboard\.

1. In the AWS SMS console, delete and restart your stuck replication jobs\.

## Server Migration Connector Fails To Connect To AWS with Error "PKIX path building failed"<a name="cert-re-signing"></a>

In some customer environments, secure network traffic is proxied through a certificate re\-signing mechanism for auditing and management purposes\. This can cause your AWS credentials to fail when the connector attempts to contact AWS SMS\. The error message contains "PKIX path building failed," indicating that an invalid certificate was presented\.

For the connector to work in such an environment, the re\-signing certificate \(a user certificate that your organization trusts and uses to sign outbound packets\) must be added to the connector's trust store, as described in the following steps\.

**To add the re\-signing certificate to the connector trust store**

1. On your connector system, disable the FreeBSD packet filter and enable SSH with the following commands: 

   ```
   sudo service pf stop
   sudo service sshd onestart
   ```

1. Copy your user certificate to the connector by a method such as the following:

   ```
   scp userCertFile ec2-user@10.0.0.100:/tmp/
   ```

1. Add the user certificate to the trust store:

   ```
   keytool -importcert -keystore /usr/local/amazon/connector/config/jetty/trustStore -storepass AwScOnNeCtOr -file /tmp/userCertFileName -alias userCertName
   ```

1. Restart services using the following command \(part of AWS Management Portal for vCenter\):

   ```
   sudo setup.rb
   ```

   Select option **3** and type "yes"\.

1. Re\-enable the packet filter: 

   ```
   sudo service pf start
   ```

## This CA Root certificate is not trusted<a name="ca-root-certificate-not-trusted"></a>

When you access the IP address of a virtual machine that you installed on\-premises, you may receive the following message:

```
This CA Root certificate is not trusted. To enable trust,
install this certificate in the Trusted Root Certifications
Authorities store.
```

You can safely ignore this message\.

## Replication Run Fails During the Preparing Stage<a name="preparing-failure"></a>

In some cases, AWS SMS allows a replication job to continue scheduling incremental replication runs even when the latest replication run has failed\. When the maximum allowed number of consecutive failures is reached, the default behavior for a replication job is to be paused\. The job can be resumed within four days, after which it is deleted\. In such cases, the Amazon EBS snapshots from the latest replication run are shared with the customer account, and a status message for the failed replication run is sent\. The message contains the snapshot IDs and states the reason for the failure\. A typical status message resembles the following:

```
EBS snapshot(s) created with snapshot ID(s): snap-12345678abcdefgh. Another run  
has been scheduled after the last run failed due to an import failure. 2 re-try run(s) 
remaining before the job will be failed.
```

The reason for replication\-run failures \(including first\-boot failures\) often correlates closely with failures observed when Amazon EC2 VM Import/Export is used for VM migrations\. For more information, see [Troubleshooting VM Import/Export](https://docs.aws.amazon.com/vm-import/latest/userguide/vmimport-troubleshooting.html)\.

If you need further help with resolving a problem, contact AWS Support\. EBS snapshots generated during a failed migration are shared with your account, and the snapshot IDs are included in the status message for the replication job\. Be sure to have these details available when you contact AWS Support\.

## Replicated AMI Doesn't Support Some Instance Types for Launch<a name="unavailable-instance-types"></a>

Some instances require ENA support\. If the migration does not enable ENA support, then the replicated AMI does not allow you to launch instances that require ENA support\.

Verify that ENA is enabled\. For more information, see [Enabling Enhanced Networking on Windows](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/enhanced-networking-ena.html#enable-enhanced-networking-ena-WIN) or [Enabling Enhanced Networking on Linux](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/enhanced-networking-ena.html) in the Amazon EC2 documentation\.

## Failure to upload to Amazon S3<a name="failure-uploading-base-disks"></a>

The following error message indicates that the vCenter cannot export the VMDK\.

```
Error message: ServerError: Failed to upload base disk(s) to S3. Please try again. 
If this problem persists, please contact AWS support: java.io.IOException: Server 
returned HTTP response code: 500 for URL: https://<url>
```

Verify that the VMDK is snapshottable and that the VM does not have any mounted ISOs\.

## Incremental Replication Delta Exceeds 1 TB<a name="delta-migrations"></a>

The connector is designed to handle frequent replication with small deltas\. The connector does not support deltas larger than 1 TB\. If you do not replicate on a regular basis, the delta can exceed this limit and the replication run fails\.

To prevent this issue, set up frequent incremental replication runs\. If you cannot replicate frequently, you can increase the delta upload limit\. For example, run the following commands on the connector to increase the part size of S3 uploads from 25 MB to 100 MB\. When prompted, select option 3\.

```
sudo sms-connector-config -set slotSizeMB 100
sudo setup.rb
```

Increasing the upload limit impacts the performance and memory usages of the connector\. Do not increase the upload limit while the connector is uploading multiple deltas\.