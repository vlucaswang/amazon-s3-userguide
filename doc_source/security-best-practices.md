# Security best practices for Amazon S3<a name="security-best-practices"></a>

Amazon S3 provides a number of security features to consider as you develop and implement your own security policies\. The following best practices are general guidelines and don’t represent a complete security solution\. Because these best practices might not be appropriate or sufficient for your environment, treat them as helpful considerations rather than prescriptions\. 

**Topics**
+ [Amazon S3 security best practices](#security-best-practices-prevent)
+ [Amazon S3 Monitoring and auditing best practices](#security-best-practices-detect)

## Amazon S3 security best practices<a name="security-best-practices-prevent"></a>

The following best practices for Amazon S3 can help prevent security incidents\.

**Disable access control lists \(ACLs\)**  
S3 Object Ownership is an Amazon S3 bucket\-level setting that you can use to control ownership of objects uploaded to your bucket and to disable or enable ACLs\. By default, Object Ownership is set to the Bucket owner enforced setting and all ACLs are disabled\. When ACLs are disabled, the bucket owner owns all the objects in the bucket and manages access to data exclusively using access management policies\.   
A majority of modern use cases in Amazon S3 no longer require the use of [access control lists \(ACLs\)](acl-overview.md), and we recommend that you keep ACLs disabled except in unusual circumstances where you must control access for each object individually\. When you disable ACLs, you can more easily maintain a bucket with objects uploaded by different AWS accounts\. Access control for your data is based on policies, such as IAM policies and S3 bucket policies\.  
Disabling ACLs simplifies permissions management\. When you create a bucket, ACLs are automatically disabled\. You can also disable ACLs on existing buckets\. In the case of an existing bucket that already has objects in it, after you disable ACLs, the object and bucket ACLs are no longer part of an access evaluation, and access is granted or denied on the basis of policies\.   
Before you disable ACLs, we recommend that you review your bucket policy to ensure that it covers all the ways that you intend to grant access to your bucket outside of your account\. After you disable ACLs, your bucket accepts only PUT requests that do not specify an ACL or `PUT` requests with bucket owner full control ACLs, such as the `bucket-owner-full-control` canned ACL or equivalent forms of this ACL expressed in XML\. Existing applications that support bucket owner full control ACLs see no impact\. `PUT` requests that contain other ACLs \(for example, custom grants to certain AWS accounts\) fail and return a `400` error with the error code `AccessControlListNotSupported`\.   
For more information\. see [Controlling ownership of objects and disabling ACLs for your bucket](about-object-ownership.md)\.

**Ensure that your Amazon S3 buckets use the correct policies and are not publicly accessible**  
Unless you explicitly require anyone on the internet to be able to read or write to your S3 bucket, you should ensure that your S3 bucket is not public\. The following are some of the steps you can take:  
+ You can use Amazon S3 Block Public Access to manage public access to Amazon S3 resources\. Block Public Access provides settings to help you avoid inadvertent public exposure of your resources\. You can apply these settings to individual access points, buckets, or entire AWS accounts\. By default, all Block Public Access settings are enabled for new buckets\. For more information, see [Blocking public access to your Amazon S3 storage](access-control-block-public-access.md)\.
+ Identify Amazon S3 bucket policies that allow a wildcard identity such as Principal “\*” \(which effectively means “anyone”\) or allows a wildcard action “\*” \(which effectively allows the user to perform any action in the Amazon S3 bucket\)\.
+ Similarly, note Amazon S3 bucket access control lists \(ACLs\) that provide read, write, or full\-access to “Everyone” or “Any authenticated AWS user\.” 
+ Use the `ListBuckets` API to scan all of your Amazon S3 buckets\. Then use `GetBucketAcl`, `GetBucketWebsite`, and `GetBucketPolicy` to determine whether the bucket has compliant access controls and configuration\.
+ Use [AWS Trusted Advisor](https://docs.aws.amazon.com/awssupport/latest/user/getting-started.html#trusted-advisor) to inspect your Amazon S3 implementation\.
+ Consider implementing on\-going detective controls using the [s3\-bucket\-public\-read\-prohibited](https://docs.aws.amazon.com/config/latest/developerguide/s3-bucket-public-read-prohibited.html) and [s3\-bucket\-public\-write\-prohibited](https://docs.aws.amazon.com/config/latest/developerguide/s3-bucket-public-write-prohibited.html) managed AWS Config Rules\.
For more information, see [Identity and access management in Amazon S3](s3-access-control.md)\. 

**Implement least privilege access**  
When granting permissions, you decide who is getting what permissions to which Amazon S3 resources\. You enable specific actions that you want to allow on those resources\. Therefore you should grant only the permissions that are required to perform a task\. Implementing least privilege access is fundamental in reducing security risk and the impact that could result from errors or malicious intent\.   
The following tools are available to implement least privilege access:  
+ [Amazon S3 actions](using-with-s3-actions.md) and [Permissions Boundaries for IAM Entities](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html)
+ [Bucket policies and user policies](using-iam-policies.md)
+ [Access control list \(ACL\) overview](acl-overview.md)
+ [Service Control Policies](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scp.html)
For guidance on what to consider when choosing one or more of the preceding mechanisms, see [Access policy guidelines](access-policy-alternatives-guidelines.md)\.

**Use IAM roles for applications and AWS services that require Amazon S3 access**  
 For applications on Amazon EC2 or other AWS services to access Amazon S3 resources, they must include valid AWS credentials in their AWS API requests\. You should not store AWS credentials directly in the application or Amazon EC2 instance\. These are long\-term credentials that are not automatically rotated and could have a significant business impact if they are compromised\.  
Instead, you should use an IAM role to manage temporary credentials for applications or services that need to access Amazon S3\. When you use a role, you don't have to distribute long\-term credentials \(such as a user name and password or access keys\) to an Amazon EC2 instance or AWS service such as AWS Lambda\. The role supplies temporary permissions that applications can use when they make calls to other AWS resources\.  
For more information, see the following topics in the *IAM User Guide*:  
+ [IAM Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)
+ [Common Scenarios for Roles: Users, Applications, and Services](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_common-scenarios.html)

   

**Consider encryption of data at rest**  
You have the following options for protecting data at rest in Amazon S3:  
+ **Server\-Side Encryption** – Request Amazon S3 to encrypt your object before saving it on disks in its data centers and then decrypt it when you download the objects\. Server\-side encryption can help reduce risk to your data by encrypting the data with a key that is stored in a different mechanism than the mechanism that stores the data itself\. 

  Amazon S3 provides these server\-side encryption options:
  + Server\-side encryption with Amazon S3‐managed keys \(SSE\-S3\)\.
  + Server\-side encryption with KMS key stored in AWS Key Management Service \(SSE\-KMS\)\.
  + Server\-side encryption with customer\-provided keys \(SSE\-C\)\.

  For more information, see [Protecting data using server\-side encryption](serv-side-encryption.md)\.
+ **Client\-Side Encryption** – Encrypt data client\-side and upload the encrypted data to Amazon S3\. In this case, you manage the encryption process, the encryption keys, and related tools\. As with server\-side encryption, client\-side encryption can help reduce risk by encrypting the data with a key that is stored in a different mechanism than the mechanism that stores the data itself\. 

  Amazon S3 provides multiple client\-side encryption options\. For more information, see [Protecting data by using client\-side encryption](UsingClientSideEncryption.md)\.

**Enforce encryption of data in transit**  
You can use HTTPS \(TLS\) to help prevent potential attackers from eavesdropping on or manipulating network traffic using person\-in\-the\-middle or similar attacks\. You should allow only encrypted connections over HTTPS \(TLS\) using the [aws:SecureTransport](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html#Conditions_Boolean) condition on Amazon S3 bucket policies\.  
Also consider implementing on\-going detective controls using the [s3\-bucket\-ssl\-requests\-only](https://docs.aws.amazon.com/config/latest/developerguide/s3-bucket-ssl-requests-only.html) managed AWS Config rule\. 

**Consider S3 Object Lock**  
[Using S3 Object Lock](object-lock.md) enables you to store objects using a "Write Once Read Many" \(WORM\) model\. S3 Object Lock can help prevent accidental or inappropriate deletion of data\. For example, you could use S3 Object Lock to help protect your AWS CloudTrail logs\.

**Enable versioning**  
Versioning is a means of keeping multiple variants of an object in the same bucket\. You can use versioning to preserve, retrieve, and restore every version of every object stored in your Amazon S3 bucket\. With versioning, you can easily recover from both unintended user actions and application failures\.   
Also consider implementing on\-going detective controls using the [s3\-bucket\-versioning\-enabled](https://docs.aws.amazon.com/config/latest/developerguide/s3-bucket-versioning-enabled.html) managed AWS Config rule\.  
For more information, see [Using versioning in S3 buckets](Versioning.md)\. 

**Consider Amazon S3 cross\-region replication**  
Although Amazon S3 stores your data across multiple geographically diverse Availability Zones by default, compliance requirements might dictate that you store data at even greater distances\. Cross\-region replication \(CRR\) allows you to replicate data between distant AWS Regions to help satisfy these requirements\. CRR enables automatic, asynchronous copying of objects across buckets in different AWS Regions\. For more information, see [Replicating objects](replication.md)\.  
CRR requires that both source and destination S3 buckets have versioning enabled\.
Also consider implementing on\-going detective controls using the [s3\-bucket\-replication\-enabled](https://docs.aws.amazon.com/config/latest/developerguide/s3-bucket-replication-enabled.html) managed AWS Config rule\.

**Consider VPC endpoints for Amazon S3 access**  
A VPC endpoint for Amazon S3 is a logical entity within an virtual private cloud \(VPC\) that allows connectivity only to Amazon S3\. You can use Amazon S3 bucket policies to control access to buckets from specific VPC endpoints, or specific VPCs\. A VPC endpoint can help prevent traffic from potentially traversing the open internet and being subject to open internet environment\.  
VPC endpoints for Amazon S3 provide multiple ways to control access to your Amazon S3 data:  
+ You can control the requests, users, or groups that are allowed through a specific VPC endpoint\.
+ You can control which VPCs or VPC endpoints have access to your S3 buckets by using S3 bucket policies\.
+ You can help prevent data exfiltration by using a VPC that does not have an internet gateway\.
For more information, see [Controlling access from VPC endpoints with bucket policies](example-bucket-policies-vpc-endpoint.md)\. 

** Use managed AWS services to receive actionable findings in your AWS accounts**  
Managed AWS services can be enabled and integrated across multiple accounts\. After you receive your findings, take action according to your incident response policy\. For each finding, determine what your required response actions will be\.  
To receive actionable findings in your AWS accounts, you can enable one of these managed AWS services:  
+ AWS Security Hub, which gives you aggregated visibility into your security and compliance status across multiple AWS accounts\. With Security Hub, you can perform security best practice checks, aggregate alerts, and automate remediation\. You can create custom actions, which allow a customer to manually invoke a specific response or remediation action on a specific finding\. You can then send custom actions to Amazon CloudWatch Events as a specific event pattern, allowing you to create a CloudWatch Events rule that listens for these actions and sends them to a target service, such as a Lambda function or Amazon SQSqueue\. You can then export findings to an Amazon S3 bucket, and share them in a standardized format for multiple use cases across AWS services\.
+ Amazon GuardDuty, which monitors object\-level API operations to identify potential security risks for data within your S3 buckets\. GuardDuty monitors threats against your Amazon S3 resources by analyzing CloudTrail management events and CloudTrail Amazon S3 data events\. These data sources monitor different kinds of activity, for example, data events for Amazon S3 include object\-level API operations, such as `GetObject`, `ListObjects`, and `PutObject`\. For more information, see [Amazon S3 protection in Amazon GuardDuty](https://docs.aws.amazon.com/guardduty/latest/ug/s3-protection.html) in the *Amazon GuardDuty User Guide;*\.
+ AWS Identity and Access ManagementAccess Analyzer, which reviews access to your internal AWS resources and validate where you've shared access outside of your AWS accounts\. With Access Analyzer for Amazon S3, you're alerted when your buckets are configured to allow access to anyone on the internet or other AWS accounts, including accounts outside of your organization\. For each public or shared bucket, you'll receive findings into the source and level of public or shared access\. For example, Access Analyzer for Amazon S3 might show that a bucket has read or write access provided through a bucket access control list \(ACL\), a bucket policy, a Multi\-Region Access Point policy, or an access point policy\. With these findings, you can take immediate and precise corrective action to restore your bucket access to what you intended\. For more information, see [Reviewing bucket access using Access Analyzer for Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-analyzer.html)\.

## Amazon S3 Monitoring and auditing best practices<a name="security-best-practices-detect"></a>

The following best practices for Amazon S3 can help detect potential security weaknesses and incidents\.

**Identify and audit all your Amazon S3 buckets**  
Identification of your IT assets is a crucial aspect of governance and security\. You need to have visibility of all your Amazon S3 resources to assess their security posture and take action on potential areas of weakness\.  
Use Tag Editor to identify security\-sensitive or audit\-sensitive resources, then use those tags when you need to search for these resources\. For more information, see [Searching for Resources to Tag](https://docs.aws.amazon.com/ARG/latest/userguide/tag-editor.html)\.   
Use Amazon S3 Inventory to audit and report on the replication and encryption status of your objects for business, compliance, and regulatory needs\. For more information, see [Amazon S3 Inventory](storage-inventory.md)\.  
Create resource groups for your Amazon S3 resources\. For more information, see [What Is AWS Resource Groups?](https://docs.aws.amazon.com/ARG/latest/userguide/welcome.html) 

**Implement monitoring using AWS monitoring tools**  
Monitoring is an important part of maintaining the reliability, security, availability, and performance of Amazon S3 and your AWS solutions\. AWS provides several tools and services to help you monitor Amazon S3 and your other AWS services\. For example, you can monitor CloudWatch metrics for Amazon S3, particularly `PutRequests`, `GetRequests`, `4xxErrors`, and `DeleteRequests`\. For more information, see [Monitoring metrics with Amazon CloudWatch](cloudwatch-monitoring.md) and, [Monitoring Amazon S3](monitoring-overview.md)\.  
For a second example, see [Example: Amazon S3 Bucket Activity](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudwatch-alarms-for-cloudtrail.html#cloudwatch-alarms-for-cloudtrail-s3-bucket-activity)\. This example describes how to create an Amazon CloudWatch alarm that is triggered when an Amazon S3 API call is made to PUT or DELETE bucket policy, bucket lifecycle, or bucket replication, or to PUT a bucket ACL\.

**Enable Amazon S3 server access logging**  
Server access logging provides detailed records of the requests that are made to a bucket\. Server access logs can assist you in security and access audits, help you learn about your customer base, and understand your Amazon S3 bill\. For instructions on enabling server access logging, see [Logging requests using server access logging](ServerLogs.md)\.  
Also consider implementing on\-going detective controls using the [s3\-bucket\-logging\-enabled](https://docs.aws.amazon.com/config/latest/developerguide/s3-bucket-logging-enabled.html) AWS Config managed rule\. 

**Use AWS CloudTrail**  
AWS CloudTrail provides a record of actions taken by a user, a role, or an AWS service in Amazon S3\. You can use information collected by CloudTrail to determine the request that was made to Amazon S3, the IP address from which the request was made, who made the request, when it was made, and additional details\. For example, you can identify CloudTrail entries for Put actions that impact data access, in particular `PutBucketAcl`, `PutObjectAcl`, `PutBucketPolicy`, and `PutBucketWebsite`\. When you set up your AWS account, CloudTrail is enabled by default\. You can view recent events in the CloudTrail console\. To create an ongoing record of activity and events for your Amazon S3 buckets, you can create a trail in the CloudTrail console\. For more information, see [Logging Data Events for Trails](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-data-events-with-cloudtrail.html) in the *AWS CloudTrail User Guide*\.  
When you create a trail, you can configure CloudTrail to log data events\. Data events are records of resource operations performed on or within a resource\. In Amazon S3, data events record object\-level API activity for individual buckets\. CloudTrail supports a subset of Amazon S3 object\-level API operations such as `GetObject`, `DeleteObject`, and `PutObject`\. For more information about how CloudTrail works with Amazon S3, see [Logging Amazon S3 API calls using AWS CloudTrail](cloudtrail-logging.md)\. In the Amazon S3 console, you can also configure your S3 buckets to [Enabling CloudTrail event logging for S3 buckets and objects](enable-cloudtrail-logging-for-s3.md)\.  
AWS Config provides a managed rule \(`cloudtrail-s3-dataevents-enabled`\) that you can use to confirm that at least one CloudTrail trail is logging data events for your S3 buckets\. For more information, see [https://docs.aws.amazon.com/config/latest/developerguide/cloudtrail-s3-dataevents-enabled.html](https://docs.aws.amazon.com/config/latest/developerguide/cloudtrail-s3-dataevents-enabled.html) in the *AWS Config Developer Guide*\.

**Enable AWS Config**  
Several of the best practices listed in this topic suggest creating AWS Config rules\. AWS Config enables you to assess, audit, and evaluate the configurations of your AWS resources\. AWS Config monitors resource configurations, allowing you to evaluate the recorded configurations against the desired secure configurations\. Using AWS Config, you can review changes in configurations and relationships between AWS resources, investigate detailed resource configuration histories, and determine your overall compliance against the configurations specified in your internal guidelines\. This can help you simplify compliance auditing, security analysis, change management, and operational troubleshooting\. For more information, see [Setting Up AWS Config with the Console](https://docs.aws.amazon.com/config/latest/developerguide/gs-console.html) in the *AWS Config Developer Guide*\. When specifying the resource types to record, ensure that you include Amazon S3 resources\.  
For an example of how to use AWS Config to monitor for and respond to Amazon S3 buckets that allow public access, see [How to Use AWS Config to Monitor for and Respond to Amazon S3 Buckets Allowing Public Access](https://aws.amazon.com/blogs/security/how-to-use-aws-config-to-monitor-for-and-respond-to-amazon-s3-buckets-allowing-public-access/) on the *AWS Security Blog*\. 

**Consider using Amazon Macie with Amazon S3**  
Amazon Macie is a data security and data privacy service that uses machine learning and pattern matching to help you discover, monitor, and protect sensitive data in your AWS environment\. Macie automates the discovery of sensitive data, such as personally identifiable information \(PII\) and financial data, to provide you with a better understanding of the data that your organization stores in Amazon S3\.  
Macie also provides you with an inventory of your S3 buckets, and it automatically evaluates and monitors those buckets for security and access control\. If Macie detects sensitive data or potential issues with the security or privacy of your data, it creates detailed findings for you to review and remediate as necessary\. For more information, see [What is Amazon Macie?](https://docs.aws.amazon.com/macie/latest/user/what-is-macie.html)

**Use Amazon S3 Storage Lens**  
Amazon S3 Storage Lens is a cloud\-storage analytics feature that you can use to gain organization\-wide visibility into object\-storage usage and activity\. S3 Storage Lens also analyzes metrics to deliver contextual recommendations that you can use to optimize storage costs and apply best practices for protecting your data\.   
With S3 Storage Lens, you can use metrics to generate summary insights, such as finding out how much storage you have across your entire organization or which are the fastest\-growing buckets and prefixes\. You can also use S3 Storage Lens metrics to identify cost\-optimization opportunities, implement data\-protection and access\-management best practices, and improve the performance of application workloads\. For example, you can identify buckets that don't have S3 Lifecycle rules to abort incomplete multipart uploads that are more than 7 days old\. You can also identify buckets that aren't following data\-protection best practices, such as using S3 Replication or S3 Versioning\. For more information, see [Understanding Amazon S3 Storage Lens](https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage_lens_basics_metrics_recommendations.html)\.

**Monitor AWS security advisories**  
You should regularly check security advisories posted in Trusted Advisor for your AWS account\. In particular, note warnings about Amazon S3 buckets with “open access permissions\.” You can do this programmatically using [describe\-trusted\-advisor\-checks](https://docs.aws.amazon.com/cli/latest/reference/support/describe-trusted-advisor-checks.html)\.  
Further, actively monitor the primary email address registered to each of your AWS accounts\. AWS will contact you, using this email address, about emerging security issues that might affect you\.  
AWS operational issues with broad impact are posted on the [AWS Service Health Dashboard](https://status.aws.amazon.com/)\. Operational issues are also posted to individual accounts via the Personal Health Dashboard\. For more information, see the [AWS Health Documentation](https://docs.aws.amazon.com/health/)\.