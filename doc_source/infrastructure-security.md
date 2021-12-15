--------

**Product update**

As of March 31, 2022, AWS will discontinue AWS Server Migration Service \(AWS SMS\)\. Going forward, we recommend [AWS Application Migration Service](http://aws.amazon.com/application-migration-service) \(AWS MGN\) as the primary migration service for lift\-and\-shift migrations\.

You can initiate new migration jobs in AWS SMS until January 1, 2022\. Complete these active migration projects by March 31, 2022\. For more information, see [When to Choose AWS Application Migration Service](http://aws.amazon.com/application-migration-service/when-to-choose-aws-mgn/)\.

--------

# Infrastructure security in AWS Server Migration Service<a name="infrastructure-security"></a>

As a managed service, AWS SMS is protected by the AWS global network security procedures that are described in the [Amazon Web Services: Overview of Security Processes](https://d0.awsstatic.com/whitepapers/Security/AWS_Security_Whitepaper.pdf) whitepaper\.

You use AWS published API calls to access AWS SMS through the network\. Clients must support Transport Layer Security \(TLS\) 1\.0 or later\. We recommend TLS 1\.2 or later\. Clients must also support cipher suites with perfect forward secrecy \(PFS\) such as Ephemeral Diffie\-Hellman \(DHE\) or Elliptic Curve Ephemeral Diffie\-Hellman \(ECDHE\)\. Most modern systems such as Java 7 and later support these modes\.

Additionally, requests must be signed using an access key ID and a secret access key that is associated with an IAM principal\. Or you can use the [AWS Security Token Service](https://docs.aws.amazon.com/STS/latest/APIReference/Welcome.html) \(AWS STS\) to generate temporary security credentials to sign requests\.