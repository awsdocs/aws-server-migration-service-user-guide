# Troubleshooting AWS SMS<a name="troubleshoot-sms"></a>

This section contains troubleshooting help for specific errors you may encounter when using AWS SMS\. Kindly visit the [AWS SMS Requirements](prereqs.md) page and ensure that the Server under migration is set up to meet the requirements before diving deep into the errors\. Note that in some cases, AWS SMS will allow a replication job to continue scheduling incremental replication runs even when the latest replication run has failed\. In such cases, the EBS snapshots from the latest replication run are shared with the customer account, the EBS snapshot IDs are identified in the replication job's Status Message and the reason for the failure is visible in the Status Message of the failed replication run\.

## FirstBootFailure: This import request failed because the instance failed to boot and establish network connectivity<a name="boot-failure"></a>

The replication run may fail and display above in the status message when the boot test for the migrated server in EC2 fails\. Here are some recommended follow up steps to identify the problem\.

### Windows VM Imports<a name="windows-troubleshoot"></a>
Kindly note the Limitations from the [AWS SMS Requirements](prereqs.md) page\. Please check the VM for the following:
+ Ensure the Windows Operating System version is supported based on above\.
+ Ensure enough free space on the C:\\ drive\.
+ Ensure page file settings are as recommended above\.

### Linux VM Imports<a name="linux-troubleshoot"></a>
Kindly note the Limitations from the [AWS SMS Requirements](prereqs.md) page\. Please check the VM for the following:
+ Ensure the flavor of Linux OS and kernel version are supported based on above\.

### Getting help from AWS Support<a name="support-troubleshoot"></a>
In case you need further help in identifying the problem, kindly reach out to AWS Support with the details of the replication job\. Please note that the EBS snapshots generated from the failed migration will be shared with your account and the EBS snapshot IDs are available in the Status Message of the replication job\. Please share these EBS snapshot IDs when reaching out to AWS Support\.

## Certificate Error When Uploading a VM to Amazon S3<a name="sms-cert-mismatch"></a>

The connector may fail to replicate your VM because the VM is on an ESXi host with an SSL certificate problem\. If this occurs, you see the following error message displayed in the **Latest run's status message** section: "ServerError: Failed to upload base disk\(s\) to S3\. Please try again\. If this problem persists, please contact AWS support: vSphere certificate hostname mismatch: Certificate for <*somehost\.somedomain\.com*> doesn't match any of the subject alternative names: \[*localhost\.localdomain*\]\."

You can override this ESXi host certificate problem by completing the following procedures:

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

1. On the **vCenter Service Account** page, enter the vCenter hostname, username, and password that you created in Step 3 of the [setup guide](SMS_setup.md)\.

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
