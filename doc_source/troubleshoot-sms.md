# Troubleshooting AWS SMS<a name="troubleshoot-sms"></a>

This section contains troubleshooting help for specific errors you may encounter when using AWS SMS\. Kindly visit the [AWS SMS Requirements](prereqs.md) page and ensure that the Server under migration is set up to meet the requirements before diving deep into the errors\. Note that in some cases, AWS SMS will allow a replication job to continue scheduling incremental replication runs even when the latest replication run has failed\. In such cases, the EBS snapshots from the latest replication run are shared with the customer account, the EBS snapshot IDs are identified in the replication job's Status Message and the reason for the failure is visible in the Status Message of the failed replication run\.

## FirstBootFailure: This import request failed because the instance failed to boot and establish network connectivity\.<a name="FirstBootFailures"></a>

When you receive the FirstBootFailure error message, it means that your virtual disk image was unable to perform one of the following steps:
+ Boot up\.
+ Install Amazon EC2 networking and disk drivers\.
+ Use a DHCP\-configured network interface to retrieve an IP address\.
+ For Windows servers, activate Windows using the Amazon EC2 Windows volume license\.

Kindly note the Limitations from the [AWS SMS Requirements](prereqs.md) page\. This should help identify possible causes of boot failures for Linux server migrations. Further steps are discussed below to debug Windows boot failures.

### Windows VM
The following best practices can help you to avoid Windows first boot failures:
+ **Disable anti\-virus and anti\-spyware software and firewalls** — These types of software can prevent installing new Windows services or drivers or prevent unknown binaries from running\. Software and firewalls can be re\-enabled after importing\.
+ **Do not harden your operating system** — Security configurations, sometimes called hardening, can prevent unattended installation of Amazon EC2 drivers\. There are numerous Windows configuration settings that can prevent import\. These settings can be reapplied once imported\.
+ **Disable or delete multiple bootable partitions** — If your virtual machine boots and requires you to choose which boot partition to use, the import may fail\.

This inability of the virtual disk image to boot up and establish network connectivity could be due to any of the following causes:

**TCP/IP networking and DHCP are not enabled**  
**Cause**: TCP/IP networking and DHCP must be enabled\.  
**Resolution**: Ensure that TCP/IP networking is enabled\. For more information, see "Setting up TCP/IP" in [Windows Server 2003/2003 R2 Retired Content](https://www.microsoft.com/en-US/download/details.aspx?id=53314) or [Configuring TCP/IP \(Windows Server 2008\)](http://technet.microsoft.com/en-us/library/cc731673%28v=ws.10%29.aspx) at the Microsoft TechNet website\. Ensure that DHCP is enabled\. For more information, see [What is DHCP](http://technet.microsoft.com/en-us/library/cc781008%28v=ws.10%29.aspx) at the Microsoft TechNet web site\.

**A volume that Windows requires is missing from the virtual machine**  
**Cause**: Importing a VM into Amazon EC2 only imports the boot disk, all other disks must be detached and Windows must able to boot before importing the virtual machine\. For example, Active Directory often stores the Active Directory database on the `D:\` drive\. A domain controller cannot boot if the Active Directory database is missing or inaccessible\.  
**Resolution**: Detach any secondary and network disks attached to the Windows VM before exporting\. Move any Active Directory databases from secondary drives or partitions onto the primary Windows partition\. For more information, see ["Directory Services cannot start" error message when you start your Windows\-based or SBS\-based domain controller](http://support.microsoft.com/kb/258062) at the Microsoft Support website\.

**Windows always boots into System Recovery Options**  
**Cause**: Windows can boot into System Recovery Options for a variety of reasons, including when Windows is pulled into a virtualized environment from a physical machine, also known as P2V\.  
**Resolution**: Ensure that Windows boots to a login prompt before exporting and preparing for import\. Do not import virtualized Windows instances that have come from a physical machine\.

**The virtual machine was created using a physical\-to\-virtual \(P2V\) conversion process**  
**Cause**: A P2V conversion occurs when a disk image is created by performing the Windows installation process on a physical machine and then importing a copy of that Windows installation into a VM\. VMs that are created as the result of a P2V conversion are not supported by Amazon EC2 VM import\. Amazon EC2 VM import only supports Windows images that were natively installed inside the source VM\.  
**Resolution**: Install Windows in a virtualized environment and migrate your installed software to that new VM\.

**Windows activation fails**  
**Cause**: During boot, Windows will detect a change of hardware and attempt activation\. During the import process we attempt to switch the licensing mechanism in Windows to a volume license provided by Amazon Web Services\. However, if the Windows activation process does not succeed, then the import fails\.  
**Resolution**: Ensure that the version of Windows that you are importing supports volume licensing\. Beta or preview versions of Windows might not\.

**No bootable partition found**  
**Cause**: During the import process of a virtual machine, we could not find the boot partition\.  
**Resolution**: Ensure that the disk you are importing has a boot partition\.

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
