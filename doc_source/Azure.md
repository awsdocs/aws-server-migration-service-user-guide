# Installing the Server Migration Connector on Azure<a name="Azure"></a>

This topic describes the steps for setting up AWS SMS to migrate VMs from Azure to Amazon EC2\. This information applies only to VMs hosted by Azure\. For information about migrating VMs from other environments, see [Installing the Server Migration Connector on VMware](VMware.md) and [Installing the Server Migration Connector on Hyper\-V](HyperV.md)\.

**Considerations for migration scenarios**
+ A single Server Migration Connector appliance can only migrate VMs under one subscription and one Azure Region\. 
+ Once a Server Migration Connector appliance is deployed, you cannot change its subscription or region unless you deploy another connector in the new subscription/region\.
+ AWS SMS supports deploying any number of Server Migration Connector appliance VMs to support migration from multiple Azure subscriptions and regions in parallel\.

The following procedures assume that you have created a properly configured IAM user as described in [Configure AWS SMS Permissions and Roles](permissions-roles.md)\.

**Topics**
+ [Step 1: Download the Connector Installation Script](#azure-script-deployment)
+ [Step 2: Validate the Integrity and Cryptographic Signature of the Script File](#validate-azure-script)
+ [Step 3: Run the Script](#run-azure-script)
+ [Step 4: Configure the Connector](#azure-connector-configuration)
+ [\(Alternative Procedure\) Deploy the Server Migration Connector Manually](#azure-manual)

## Step 1: Download the Connector Installation Script<a name="azure-script-deployment"></a>

AWS SMS provides a downloadable Powershell script to deploy the connector in your Azure environment\. The script is cryptographically signed by AWS\. Complete this procedure to run the Powershell script and install the connector automatically in your Azure environment\. The script requires Powershell 5\.1 or later\.

**Note**  
AWS recommends using the scripted installation, but you can alternatively install the connector manually\. For more information, see [\(Alternative Procedure\) Deploy the Server Migration Connector Manually](#azure-manual)

**To download the script and hash files**

1. Download the Powershell script and hash files from the following URLs:  
****    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/server-migration-service/latest/userguide/Azure.html)

1. After download, transfer the files to the computer or computers where you plan to run the script\.

## Step 2: Validate the Integrity and Cryptographic Signature of the Script File<a name="validate-azure-script"></a>

Before running the script, we recommend that you validate its integrity and signature, ensuring that it has not changed in transit to your computer\. These procedures assume that you have downloaded the script and the hash files, that they are installed on the desktop of the computer where you plan to run the script, and that you are signed in as administrator\. You may need to modify the procedures to match your setup\.

**To validate script integrity using cryptographic hashes \(PowerShell\)**

1. Use one or both of the downloaded hash files to validate the integrity of the script file\.

   1. To validate with the MD5 hash, run the following command in a PowerShell window:

      ```
      PS C:\Users\Administrator\Desktop> Get-FileHash aws-sms-azure-setup.ps1 -Algorithm MD5
      ```

      This should return information similar to the following:

      ```
      Algorithm         Hash
      ---------         ----
      MD5               1AABAC6D068EEF6EXAMPLEDF50A05CC8
      ```

   1. To validate with the SHA256 hash, run the following command in a PowerShell window:

      ```
      PS C:\Users\Administrator\Desktop> Get-FileHash aws-sms-azure-setup.ps1 -Algorithm SHA256
      ```

      This should return information similar to the following:

      ```
      Algorithm         Hash
      ---------         ----
      SHA256            6B86B273FF34FCE19D6B804EFF5A3F574EXAMPLE22F1D49C01E52DDB7875B4B
      ```

1. Compare the returned hash values with the values provided in the downloaded files, `aws-sms-azure-setup.ps1.md5` and `aws-sms-azure-setup.ps1.sha256`\.

Next, use either PowerShell or the Windows user interface to check that the script file includes a valid signature from AWS\.

**To check the script file for a valid cryptographic signature \(PowerShell\)**
+ In a PowerShell window, run the following command:

  ```
  PS C:\Users\Administrator\Desktop> Get-AuthenticodeSignature aws-sms-azure-setup.ps1 | Select *
  ```

  A correctly signed script file should return information similar to the following:

  ```
  SignerCertificate          : [Subject]
                              CN="Amazon Web Services, Inc." ...
                           [Issuer]
                              CN=DigiCert EV Code Signing CA (SHA2), OU=www.digicert.com, O=DigiCert Inc, C=US
  ...
  
  TimeStamperCertificate     : 
  Status                     : Valid
  StatusMessage              : Signature verified.
  Path                       : C:\Users\Administrator\Desktop\aws-sms-azure-setup.ps1
  
  ...
  ```

**To check the script file for a valid cryptographic signature \(Windows GUI\)**

1. In Windows Explorer, open the context \(right\-click\) menu on the script file and choose **Properties**, **Digital Signatures**, **Amazon Web Services**, and **Details**\.

1. Verify that the displayed information contains "This digital signature is OK" and that "Amazon Web Services, Inc\." is the signer\.

## Step 3: Run the Script<a name="run-azure-script"></a>

Complete the following steps to install the connector in your Azure environment\.

1. Run the script\.

   The script takes a Storage Account and Virtual Network as mandatory arguments\. For more information, see [Requirements for Azure connector](prereqs.md#azure-connector-requirements)\.

1. When the script prompts for an Azure login, use a login that has administrator permissions for the subscription under which you are deploying this connector\. 

1. Once the script completes, the connector is deployed in your account\. The script prints out the connector's private IP address and the Object ID of the connector VM's System Assigned Identity, which you need for registration\. 

## Step 4: Configure the Connector<a name="azure-connector-configuration"></a>

After you have deployed the connector using either the scripted or manual method, you must configure it by completing the following steps\.

**To configure the connector**

1. From another VM on the same virtual network where Server Migration Connector is deployed, open a browser and navigate to the connector's private IP address that you saved earlier\.
**Note**  
If you do not have another VM on the same virtual network as the connector, you can temporarily open port 443 on the connector VM to the public and register using its public IP\. We do not recommend this\.

1. Follow the steps to register the connector\. You will need the object ID of the connector VM's System Assigned Identity, which you saved earlier\. You can also find the object ID of this identity from the Azure Portal **Identity** pane of the connector VM\.

## \(Alternative Procedure\) Deploy the Server Migration Connector Manually<a name="azure-manual"></a>

Complete this procedure to install the connector manually in your Azure environment\.

**To install the connector manually**

1. Log into the Azure Portal as a user with administrator permissions for the subscription under which you are deploying this connector\.

1. Make sure you are ready to supply a Storage Account, its Resource Group, a Virtual Network, and the Azure Region as described in [Requirements for Azure connector](prereqs.md#azure-connector-requirements)\.

1. Download the connector VHD and associated files from the URLs in the following table\.   
****    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/server-migration-service/latest/userguide/Azure.html)

1. Verify the cryptographic integrity of the connector VHD using procedures similar to those described in [Step 2: Validate the Integrity and Cryptographic Signature of the Script File](#validate-azure-script)\.

1. Upload the connector VHD and associated files to your Storage Account\.

1. Create a new managed disk with the following parameter values:
   + **Resource Group**: Select your resource group
   + **Name**: Any name \- for example, sms\-connector\-disk\-westus
   + **Region**: Select your Azure Region
   + **Availability Zone**: None
   + **Source Type**: Storage Blob\. \(Choose the VHD blob you uploaded from step 3\.c\.\)
   + **OSType**: Linux
   + **Size**: 60 GB/Standard HDD

1. Choose **Create VM** to create a new virtual machine from the managed disk you created\. Assign the following parameter values\.

   Under the **Basics** tab
   + **Resource Group**: Type in your resource group
   +  **Virtual Machine Name**: Any name, for example sms\-connector\-vm\-westus
   + **Region**: Select your Azure Region
   + **Size**: F4s
   + **Public Inbound Ports**: None

   Under the **Disks** tab:
   + **OS Disk Type**: Standard HDD

   Under the **Networking** tab:
   + **Virtual Network**: Type in your Virtual Network name\.
   + **Subnet**: Leave as default or choose a particular subnet\.
   + **Public IP**: Leave as new
   + **NIC Network Security Group**: Basic
   + **Public Inbound Ports**: None
   + Accept defaults for the remaining fields\.

   Under the **Management** tab: 
   + **Boot Diagnostics**: On 
   + **OS Guest Diagnostics**: Off
   + **Diagnostics Storage account**: Storage Account
   + **System Assigned Managed Identity**: On
   + **Enable auto\-shutdown**: Off

1. Review and create the VM\. This will be your connector VM\.

1. Download the two role documents from this page:
   + [https://s3\.amazonaws\.com/sms\-connector/SMSConnectorRole\.json](https://s3.amazonaws.com/sms-connector/SMSConnectorRole.json)
   + [https://s3\.amazonaws\.com/sms\-connector/SMSConnectorRoleSA\.json](https://s3.amazonaws.com/sms-connector/SMSConnectorRoleSA.json)

1. **\(Important\)** Customize the role documents\.

   Edit `SMSConnectorRole.json` and change the `AssignableScopes` field to match your subscription ID\.

   Edit `SMSConnectorRoleSA.json`\. Change the `name` field to `sms-connector-role-Your Storage Account`\. For example, if your account is *testStorage*, then the name field needs to be `sms-connector-role-testStorage`\. Then change the `AssignableScopes` field to match your Subscription, Resource Group and Storage Account values\.

1. Create a role definition\. Currently, there is no way to create role definition from Azure Portal\. You will need to use Az CLI or Az Powershell for this step\. Use the [New\-AzRoleDefinition](https://docs.microsoft.com/en-us/powershell/module/az.resources/new-azroledefinition) or [az role definition create](https://docs.microsoft.com/en-us/cli/azure/role/definition#az-role-definition-create) command to create these two custom roles in your subscription using the JSON files you created above\.

1. Assign roles to the connector VM\. In Azure Portal, choose **Storage Account**, **Access Control**, **Roles**, **Add**, **Add Role Assignment**\. Choose the role `sms-connector-role`, assign access to *Virtual Machine*, and select the connector VM’s System Assigned Identity from the list\. Repeat this for the role `sms-connector-role-Your Storage Account`\.

1. Restart the connector VM to activate the role assignments\.

1. Continue to [Step 4: Configure the Connector](#azure-connector-configuration)\.