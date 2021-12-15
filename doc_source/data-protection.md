--------

**Product update**

As of March 31, 2022, AWS will discontinue AWS Server Migration Service \(AWS SMS\)\. Going forward, we recommend [AWS Application Migration Service](http://aws.amazon.com/application-migration-service) \(AWS MGN\) as the primary migration service for lift\-and\-shift migrations\.

You can initiate new migration jobs in AWS SMS until January 1, 2022\. Complete these active migration projects by March 31, 2022\. For more information, see [When to Choose AWS Application Migration Service](http://aws.amazon.com/application-migration-service/when-to-choose-aws-mgn/)\.

--------

# Data protection in AWS Server Migration Service<a name="data-protection"></a>

The AWS [shared responsibility model](http://aws.amazon.com/compliance/shared-responsibility-model/) applies to data protection in AWS Server Migration Service\. As described in this model, AWS is responsible for protecting the global infrastructure that runs all of the AWS Cloud\. You are responsible for maintaining control over your content that is hosted on this infrastructure\. This content includes the security configuration and management tasks for the AWS services that you use\. For more information about data privacy, see the [Data Privacy FAQ](http://aws.amazon.com/compliance/data-privacy-faq)\. For information about data protection in Europe, see the [AWS Shared Responsibility Model and GDPR](http://aws.amazon.com/blogs/security/the-aws-shared-responsibility-model-and-gdpr/) blog post on the *AWS Security Blog*\.

For data protection purposes, we recommend that you protect AWS account credentials and set up individual user accounts with AWS Identity and Access Management \(IAM\)\. That way each user is given only the permissions necessary to fulfill their job duties\. We also recommend that you secure your data in the following ways:
+ Use multi\-factor authentication \(MFA\) with each account\.
+ Use SSL/TLS to communicate with AWS resources\. We recommend TLS 1\.2 or later\.
+ Set up API and user activity logging with AWS CloudTrail\.
+ Use AWS encryption solutions, along with all default security controls within AWS services\.
+ Use advanced managed security services such as Amazon Macie, which assists in discovering and securing personal data that is stored in Amazon S3\.
+ If you require FIPS 140\-2 validated cryptographic modules when accessing AWS through a command line interface or an API, use a FIPS endpoint\. For more information about the available FIPS endpoints, see [Federal Information Processing Standard \(FIPS\) 140\-2](http://aws.amazon.com/compliance/fips/)\.

We strongly recommend that you never put confidential or sensitive information, such as your customers' email addresses, into tags or free\-form fields such as a **Name** field\. This includes when you work with AWS SMS or other AWS services using the console, API, AWS CLI, or AWS SDKs\. Any data that you enter into tags or free\-form fields used for names may be used for billing or diagnostic logs\. If you provide a URL to an external server, we strongly recommend that you do not include credentials information in the URL to validate your request to that server\.

## Encryption at rest<a name="encryption-rest"></a>

When replicating server volumes from your on\-premises environment, AWS SMS stores data temporarily in an intermediate S3 bucket\. After replication is complete, AWS SMS deletes this data stored Amazon S3\. Otherwise, AWS SMS does not store your data at rest\.

## Encryption in transit<a name="encryption-transit"></a>

Data in transit is encrypted using TLS\. This includes traffic from the client to the AWS SMS console, the Server Migration Connector to Amazon S3, and the Server Migration Connector to AWS SMS\.