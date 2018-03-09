# Server Migration Service \(SMS\) Requirements<a name="prereqs"></a>

Your VMware vSphere or Microsoft Hyper\-V/SCVMM environment must meet the following requirements for you to use the Server Migration Service to migrate your on\-premises virtualized servers to Amazon EC2\.

## General Requirements<a name="general-requirements"></a>

Before setting up AWS SMS, take action as needed to meet all of the following requirements\.

**All VMs**

+ Disable any antivirus or intrusion detection software on the VM you are migrating\. These services can be re\-enabled after the migration process is complete\.

+ Disconnect any CD\-ROM drives \(virtual or physical\) connected to the VM\.

**Windows VMs**

+ Enable Remote Desktop \(RDP\) for remote access\.

+ Install the appropriate version of \.NET Framework on the VM\. Note that \.NET Framework 4\.5 or later will be installed automatically on your VM if required\.  
****    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/server-migration-service/latest/userguide/prereqs.html)

+ When preparing a Microsoft Windows VM for migration, configure a fixed pagefile size and ensure that at least 6 GiB of free space is available on the root volume\. This is necessary for successful installation of the drivers\.

+ Make sure that your host firewall \(such as Windows firewall\) allows access to RDP\. Otherwise, you will not be able to access your instance after the migration is complete\.

+ Apply the following hotfixes:

  + [You cannot change system time if `RealTimeIsUniversal` registry entry is enabled in Windows](https://support.microsoft.com/en-us/help/2922223/you-cannot-change-system-time-if-realtimeisuniversal-registry-entry-is-enabled-in-windows)

  + [High CPU usage during DST changeover in Windows Server 2008, Windows 7, or Windows Server 2008 R2](https://support.microsoft.com/en-us/help/2800213/high-cpu-usage-during-dst-changeover-in-windows-server-2008,-windows-7,-or-windows-server-2008-r2.)

**Linux VMs**

+ Enable Secure Shell \(SSH\) for remote access\.

+ Make sure that your host firewall \(such as iptables\) allows access to SSH\. Otherwise, you will not be able to access your instance after the migration is complete\.

+ Make sure that your Linux VM uses GRUB \(GRUB legacy\) or GRUB 2 as its bootloader\.

+ Make sure that the root volume of your Linux VM uses one of the following file systems:

  + EXT2

  + EXT3

  + EXT4

  + Btrfs

  + JFS

  + XFS

**Programmatic Modifications to VMs**  
When importing a VM, AWS modifies the file system to make the imported VM accessible to the customer\. The following actions may occur:

+ \[Linux\] Installing Citrix PV drivers either directly in OS or modify initrd/initramfs to contain them\.

+ \[Linux\] Modifying network scripts to replace static IPs with dynamic IPs\.

+ \[Linux\] Modifying `/etc/fstab`, commenting out invalid entries and replacing device names with UUIDs\. If no matching UUID can be found for a device, the `nofail` option is added to the device description\. You will need to correct the device naming and remove `nofail` after import\. As a best practice when preparing your VMs for import, we recommend that you specify your VM disk devices by UUID rather than device name\.

  Entries in `/etc/fstab` that contain non\-standard file system types \(cifs, smbfs, vboxsf, sshfs, etc\.\) will be disabled\.

+ \[Linux\] Modifying grub bootloader settings such as the default entry and timeout\.

+ \[Windows\] Modifying registry settings to make the VM bootable\.

When writing a modified file, AWS retains the original file at the same location under a new name\.

## AWS Server Migration Connector Requirements<a name="connector_prereqs"></a>

The Server Migration Connector is a FreeBSD VM that you install in your on\-premises virtualization environment\. Its hardware and software requirements are as follows: 

**Requirements for VMware connector**

+ vCenter version 5\.1 or higher \(validated up to 6\.5\)

+ ESXi 5\.1 or higher \(validated up to 6\.5\)

+ Minimum 8 GiB RAM

+ Minimum available disk storage of 20 GiB \(thin\-provisioned\) or 250 GiB \(thick\-provisioned\)

+ Support for the following network services\. Note that you might need to reconfigure your firewall to permit stateful outbound connections from the connector to these services\.

  + DNS—Allow the connector to initiate connections to port 53 for name resolution\.

  + HTTPS on vCenter—Allow the connector to initiate secure web connections to port 443 of vCenter\. You can also configure a non\-default port at your discretion\.
**Note**  
If your vCenter Server is configured to use a non\-default port, enter both the vCenter's hostname and port, separated by a colon \(for example, `HOSTNAME:PORT` or `IP:PORT`\) in the vCenter Service Account page in **Connector setup**\.

  +  HTTPS on ESXi—Allow the connector to initiate secure web connections to port 443 of the ESXi hosts containing the VMs you intend to migrate\.

  + ICMP—Allow the connector to initiate connections using ICMP\.

  + NTP—The connector must be able to reach a time server on port 123\.

+ Allow outbound connections from the connector to the following URL ranges: 

  + \*\.amazonaws\.com

  + \*\.aws\.amazon\.com

  + \*\.ntp\.org \(Optional; used only to validate that connector time is in sync with NTP\.\)

**Requirements for Hyper\-V connector**

+ Hyper\-V role on Windows Server 2012 R2 or Windows Server 2016

+ Active Directory 2012 or above

+ \[Optional\] SCVMM 2012 SP1 or SCVMM 2016

+ Minimum 8 GiB RAM

+ Minimum available disk storage of 300 GiB

+ Support for the following network services\. Note that you might need to reconfigure your firewall to permit stateful outbound connections from the connector to these services\.

  + DNS—Allow the connector to initiate connections to port 53 for name resolution\.

  + HTTPS on WinRM port 5986 on your SCVMM or standalone Hyper\-V host

  + Inbound HTTPS on port 443 of the connector—Allow the connector to receive secure web connections on port 443 from Hyper\-V hosts containing the VMs you intend to migrate\.

  + ICMP—Allow the connector to initiate connections using ICMP\.

  + NTP—The connector must be able to reach a time server on port 123\.

+ Allow outbound connections from the connector to the following URL ranges: 

  + \*\.amazonaws\.com

  + \*\.aws\.amazon\.com

  + \*\.ntp\.org \(Optional; used only to validate that connector time is in sync with NTP\.\)

## Operating Systems Supported by AWS SMS<a name="os_prereqs"></a>

The following operating systems can be migrated to EC2 using SMS:

**Windows \(32\- and 64\-bit\)**

+ Microsoft Windows Server 2003 \(Standard, Datacenter, Enterprise\) with Service Pack 1 \(SP1\) or later \(32\- and 64\-bit\)

+ Microsoft Windows Server 2003 R2 \(Standard, Datacenter, Enterprise\) \(32\- and 64\-bit\)

+ Microsoft Windows Server 2008 \(Standard, Datacenter, Enterprise\) \(32\- and 64\-bit\)

+ Microsoft Windows Server 2008 R2 \(Standard, Datacenter, Enterprise\) \(64\-bit only\)

+ Microsoft Windows Server 2012 \(Standard, Datacenter\) \(64\-bit only\)

+ Microsoft Windows Server 2012 R2 \(Standard, Datacenter\) \(64\-bit only\) \(Nano Server installation not supported\)

+ Microsoft Windows Server 2016 \(Standard, Datacenter\) \(64\-bit only\)

+ Microsoft Windows 7 \(Professional, Enterprise, Ultimate\) \(US English\) \(32\- and 64\-bit\)

+ Microsoft Windows 8 \(Professional, Enterprise\) \(US English\) \(32\- and 64\-bit\)

+ Microsoft Windows 8\.1 \(Professional, Enterprise\) \(US English\) \(64\-bit only\)

+ Microsoft Windows 10 \(Professional, Enterprise, Education\) \(US English\) \(64\-bit only\)

**Linux/Unix \(64\-bit\)**

+ Ubuntu 12\.04, 12\.10, 13\.04, 13\.10, 14\.04, 14\.10, 15\.04, 16\.04, 16\.10

+ Red Hat Enterprise Linux \(RHEL\) 5\.1\-5\.11, 6\.1\-6\.9, 7\.0\-7\.3 \(6\.0 lacks required drivers\)
**Note**  
See [Limitations](#limitations) for additional information about RHEL 5\.x support\.

+ SUSE Linux Enterprise Server 11 with Service Pack 1 and kernel 2\.6\.32\.12\-0\.7

+ SUSE Linux Enterprise Server 11 with Service Pack 2 and kernel 3\.0\.13\-0\.27

+ SUSE Linux Enterprise Server 11 with Service Pack 3 and kernel 3\.0\.76\-0\.11, 3\.0\.101\-0\.8, or 3\.0\.101\-0\.15

+ SUSE Linux Enterprise Server 11 with Service Pack 4 and kernel 3\.0\.101\-63

+ SUSE Linux Enterprise Server 12 with kernel 3\.12\.28\-4

+ SUSE Linux Enterprise Server 12 with Service Pack 1 and kernel 3\.12\.49\-11

+ CentOS 5\.1\-5\.11, 6\.1\-6\.6, 7\.0\-7\.3 \(6\.0 lacks required drivers\)

+ Debian 6\.0\.0\-6\.0\.8, 7\.0\.0\-7\.8\.0, 8\.0\.0

+ Oracle Linux 6\.1\-6\.6, 7\.0\-7\.1

+ Fedora Server 19\-21

## Volume Types and File Systems Supported by AWS SMS<a name="volume-types-file-systems"></a>

AWS Server Migration Service supports migrating Windows and Linux instances with the following file systems:

**Windows \(32– and 64\-bit\)**  
MBR\-partitioned volumes that are formatted using the NTFS file system\. GUID Partition Table \(GPT\) partitioned volumes are not supported\.

**Linux/Unix \(64\-bit\)**  
MBR\-partitioned volumes that are formatted using the ext2, ext3, ext4, Btrfs, JFS, or XFS file system\. GUID Partition Table \(GPT\) partitioned volumes are not supported\.

AMIs with volumes using EBS encryption are not supported\.

## Licensing Options<a name="licensing"></a>

When you create a new replication job, the AWS Server Migration Service console provides a **License type** option\. The possible values include:

+ **Auto** \(default\)

  Detects the source\-system operating system \(OS\) and applies the appropriate license to the migrated virtual machine \(VM\)\.

+ **AWS**

  Replaces the source\-system license with an AWS license, if appropriate, on the migrated VM\.

+ **BYOL**

  Retains the source\-system license, if appropriate, on the migrated VM\.

**Note**  
If you choose a license type that is incompatible with your VM, the replication job fails with an error message\. For more information, see the OS\-specific information below\.

The same licensing options are available through the AWS SMS API and CLI\. For example:

```
aws sms create-replication-job --license-type <value>
```

The value of the `--license-type` parameter can be AWS or BYOL\. Leaving it unset is the same as choosing **Auto** in the console\. 

### Licensing for Linux<a name="linux"></a>

Linux operating systems support only BYOL licenses\. Choosing **Auto** \(the default\) means that AWS SMS uses a BYOL license\.

Migrated Red Hat Enterprise Linux \(RHEL\) VMs must use Cloud Access \(BYOL\) licenses\. For more information, see [Red Hat Cloud Access](https://www.redhat.com/en/technologies/cloud-computing/cloud-access) on the Red Hat website\.

Migrated SUSE Linux Enterprise Server VMs must use SUSE Public Cloud Program \(BYOS\) licenses\. For more information, see [SUSE Public Cloud Program—Bring Your Own Subscription](https://www.suse.com/docrep/documents/h4jlnjpfx3/suse_public_cloud_program_bring_your_own_subscription_faq_external.pdf)\.

### Licensing for Windows<a name="windows"></a>

Windows server operating systems support either BYOL or AWS licenses\. Windows client operating systems \(such as Windows 10\) support only BYOL licenses\. 

If you choose **Auto** \(the default\), AWS SMS uses the AWS license if the VM has a server OS\. Otherwise, the BYOL license is used\. 

The following rules apply when you use your BYOL Microsoft license, either through MSDN or [Windows Software Assurance Per User](http://download.microsoft.com/download/5/c/7/5c727885-ec15-4920-818b-4d140ec6c38a/Windows_SA_per_User_at_a_Glance.pdf):

+ Your BYOL instances are priced at the prevailing Amazon EC2 Linux instance pricing, provided that you meet the following conditions: 

  + Run on a Dedicated Host \([Dedicated Hosts](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/dedicated-hosts-overview.html)\)

  + Launch from VMs sourced from software binaries provided by you using AWS SMS, which are subject to the current terms and abilities of AWS SMS

  + Designate the instances as BYOL instances

  + Run the instances within your designated AWS regions, and where AWS offers the BYOL model

  + Activate using Microsoft keys that you provide or which are used in your key management system

+ You must account for the fact that when you start an Amazon EC2 instance, it can run on any one of many servers within an Availability Zone\. This means that each time you start an Amazon EC2 instance \(including a stop/start\), it may run on a different server within an Availability Zone\. You must account for this fact in light of the limitations on license reassignment as described in the Microsoft Volume Licensing Product Use Rights \(PUR\)/Product Terms \(PT\) available at [Volume Licensing for Microsoft Products and Online Services](http://www.microsoft.com/licensing/about-licensing/product-licensing.aspx), or consult your specific use rights to determine if your rights are consistent with this usage\.

+ You must be eligible to use the BYOL program for the applicable Microsoft software under your agreements with Microsoft, for example, under your MSDN user rights or under your Windows Software Assurance Per User Rights\. You are solely responsible for obtaining all required licenses and for complying with all applicable Microsoft licensing requirements, including the PUR/PT\. Further, you must have accepted Microsoft's End User License Agreement \(Microsoft EULA\), and by using the Microsoft Software under the BYOL program, you agree to the Microsoft EULA\.

+ AWS recommends that you consult with your own legal and other advisers to understand and comply with the applicable Microsoft licensing requirements\. Usage of the Services \(including usage of the **licenseType** parameter and **BYOL** flag\) in violation of your agreements with Microsoft is not authorized or permitted\.

## Limitations<a name="limitations"></a>

+ When migrating VMs managed by Hyper\-V/SCVMM, SMS supports only Generation 1 VMs \(using either VHD or VHDX disk format\)\.

+ AWS SMS does not support VMs on Hyper\-V running any version of RHEL 5 if backed by a VHDX disk\. We recommend converting disks in this format to VHD for migration\.

+ A VM's boot volume must use Master Boot Record \(MBR\) partitions and cannot exceed 2 TiB \(uncompressed\) due to MBR limitations\. Additional non\-bootable volumes may use GUID Partition Table \(GPT\) partitioning but cannot be bigger than 4 TiB\.

+ An imported VM may fail to boot if the root partition is not on the same virtual hard drive as the MBR\.

+ An SMS replication job will fail for VMs with more than 22 volumes attached\.

+ AMIs with volumes using EBS encryption are not supported\.

+ On VMware, AWS SMS does not support VMs that use Raw Device Mapping \(RDM\)\. Only VMDK disk images are supported\.

+ A migrated VM may fail to boot if the root partition is not on the same virtual hard disk as the MBR\.

+ AWS SMS creates AMIs that use Hardware Virtual Machine \(HVM\) virtualization\. It can't create AMIs that use Paravirtual \(PV\) virtualization\. Linux PVHVM drivers are supported within migrated VMs\.

+ Migrated Linux VMs must use 64\-bit images\. Migrating 32\-bit Linux images is not supported\.

+ Migrated Linux VMs should use default kernels for best results\. VMs that use custom Linux kernels might not migrate successfully\.

+ When preparing Amazon EC2 Linux VMs for migration, make sure that at least 250 MiB of disk space is available on the root volume for installing drivers and other software\. For Microsoft Windows VMs, configure a fixed pagefile size and ensure that at least 6 GiB of free space is available on the root volume\.

+ Multiple network interfaces are not currently supported\. After migration, your VM will have a single virtual network interface that uses DHCP to assign addresses\. Your instance receives a private IP address\.

+ A VM migrated into a VPC does not receive a public IP address, regardless of the auto\-assign public IP setting for the subnet\. Instead, you can allocate an Elastic IP address to your account and associate it with your instance\.

+ Internet Protocol version 6 \(IPv6\) IP addresses are not supported\.

+ VMs that are created as the result of a P2V conversion are not supported\. A P2V conversion occurs when a disk image is created by performing a Linux or Windows installation process on a physical machine and then importing a copy of that Linux or Windows installation to a VM\.

+ AWS SMS does not install the single root I/O virtualization \(SR\-IOV\) drivers except with imports of Microsoft Windows Server 2012 R2 VMs\. These drivers are not required unless you plan to use enhanced networking, which provides higher performance \(packets per second\), lower latency, and lower jitter\. For Microsoft Windows Server 2012 R2 VMs, SR\-IOV drivers are automatically installed as a part of the migration process\.

+ AWS SMS does not currently support VMware SEsparse delta\-file format\. 

+ Because independent disks are unaffected by snapshots, AWS SMS does not support interval replication for VMDKs in independent mode\.

+ Windows language packs that use UTF\-16 \(or non\-ASCII\) characters are not supported for import\. We recommend using the English language pack when importing Windows Server 2003, Windows Server 2008, and Windows Server 2012 R1 VMs\.

+ AWS SMS does not support VMs that have a mix of VHD and VHDX disk files\.

## Other Requirements<a name="other_prereqs"></a>

**Support for VMware vMotion**

AWS Server Migration Service partially supports vMotion, Storage vMotion, and other features based on virtual machine migration \(such as DRS and Storage DRS\) subject to the following limitations:

+ Migrating a virtual machine to a new ESXi host or datastore after one replication run ends, and before the next replication run begins, is supported as long as the Server Migration Connector's vCenter service account has sufficient permissions on the destination ESXi host, datastores, and datacenter, and on the virtual machine itself at the new location\.

+ Migrating a virtual machine to a new ESXi host, datastore, and/or datacenter while a replication run is active—that is, while a virtual machine upload is in progress—is not supported\. 

+ Cross vCenter vMotion is not supported for use with the AWS Server Migration Service\.

**Support for VMware vSAN**  
VMs on vSAN datastores are only supported when **Replication job type** on the **Configure replication jobs settings** page is set to **One\-time migration**\.

**Support for VMware Virtual Volumes \(VVol\)**  
AWS does not provide support for migrating VMware Virtual Volumes\. Some implementations may work, however\.