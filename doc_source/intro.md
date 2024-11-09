# What is AWS Secrets Manager?<a name="intro"></a>

* application's credentials | history
  * embed the credentials | application
  * when rotation credentials came -> update the application / use the new credentials
  * when applications distributed (-> multiple applications) -> you needed to update ALL
    * -> by laziness, NO rotations are done

* AWS Secrets Manager
  * -- enables you to -- replace hardcoded credentials | your code
    * 👀-- via -- API call to Secrets Manager 👀
      * benefits
        * if someone examine your code -> secret can NOT be compromised
        * automatic rotation can be configured

* see [Get started with AWS Secrets Manager](getting-started.md)

**Topics**
+ [Basic AWS Secrets Manager scenario](#intro-basic-scenario)
+ [Features of AWS Secrets Manager](#features)
+ [Compliance with standards for AWS Secrets Manager](#asm_compliance)
+ [Pricing for AWS Secrets Manager](#asm_pricing)
+ [Support and feedback for AWS Secrets Manager](#support-and-feedback)

## Basic AWS Secrets Manager scenario<a name="intro-basic-scenario"></a>

* goal
  * MOST basic scenario
* scenario
  * credentials to access to a database
    * stored | Secrets Manager
    * used by an application

        ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/secretsmanager/latest/userguide/images/ASM-Basic-Scenario.png)

1. The database administrator -- creates a -- set of credentials | Personnel database /
   1. used by an application -- MyCustomApp
   2. configured / the application -- can access to the -- Personnel database
2. The database administrator -- stores the credentials as a -- secret | Secrets Manager (named *MyCustomAppCreds*\)
   1. -> Secrets Manager 
      1. encrypts
      2. stores the credentials -- as the -- *protected secret text*\.
3. if MyCustomApp wants to access the database -> the application -- queries -- Secrets Manager
4. Secrets Manager 
   1. retrieves the secret,
   2. decrypts the protected secret text
   3. returns the secret -- via HTTPS with TLS, to the -- client app
5. The client application
   1. parses the credentials,
   2. connection string,
   3. uses it, -- to access the -- database server

## Features of AWS Secrets Manager<a name="features"></a>
* TODO:
### Programmatically retrieve encrypted secret values at runtime<a name="features_retrieve-by-label"></a>

Secrets Manager helps you improve your security posture by removing hard\-coded credentials from your application source code, and by not storing credentials within the application, in any way\. Storing the credentials in or with the application subjects them to possible compromise by anyone who can inspect your application or the components\. Since you have to update your application and deploy the changes to every client before you can deprecate the old credentials, this process makes rotating your credentials difficult\. 

Secrets Manager enables you to replace stored credentials with a runtime call to the Secrets Manager Web service, so you can retrieve the credentials dynamically when you need them\. 

Most of the time, your client requires access to the most recent version of the encrypted secret value\. When you query for the encrypted secret value, you can choose to provide only the secret name or Amazon Resource Name \(ARN\), without specifying any version information at all\. If you do this, Secrets Manager automatically returns the most recent version of the secret value\.

However, other versions can exist at the same time\. Most systems support secrets more complicated than a simple password, such as full sets of credentials including the connection details, the user ID, and the password\. Secrets Manager allows you to store multiple sets of these credentials at the same time\. Secrets Manager stores each set in a different version of the secret\. During the secret rotation process, Secrets Manager tracks the older credentials, as well as the new credentials you want to start using, until the rotation completes\.

### Store different types of secrets<a name="features_storing-secrets"></a>

Secrets Manager enables you to store text in the encrypted secret data portion of a secret\. This typically includes the connection details of the database or service\. These details can include the server name, IP address, and port number, as well as the user name and password used to sign in to the service\. For details on secrets, see the [maximum and minimum values](reference_limits.html#reference_limits_max-min)\. The protected text doesn't include:
+  Secret name and description
+  Rotation or expiration settings
+  ARN of the KMS key associated with the secret
+ Any attached AWS tags 

### Encrypt your secret data<a name="features_kms-encryption"></a>

Secrets Manager encrypts the protected text of a secret by using [AWS Key Management Service \(AWS KMS\)](https://docs.aws.amazon.com/kms/latest/developerguide/)\. Many AWS services use AWS KMS for key storage and encryption\. AWS KMS ensures secure encryption of your secret when at rest\. Secrets Manager associates every secret with a KMS key\. It can be either AWS managed key for Secrets Manager for the account \(`aws/secretsmanager`\), or a customer managed key you create in AWS KMS\. 

Whenever Secrets Manager encrypt a new version of the protected secret data, Secrets Manager requests AWS KMS to generate a new data key from the KMS key\. Secrets Manager uses this data key for [envelope encryption](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#enveloping)\. Secrets Manager stores the encrypted data key with the protected secret data\. Whenever the secret needs decryption, Secrets Manager requests AWS KMS to decrypt the data key, which Secrets Manager then uses to decrypt the protected secret data\. Secrets Manager never stores the data key in unencrypted form, and always disposes the data key immediately after use\.

In addition, Secrets Manager, by default, only accepts requests from hosts using open standard [Transport Layer Security \(TLS\)](https://en.wikipedia.org/wiki/Transport_Layer_Security) and [Perfect Forward Secrecy](https://en.wikipedia.org/wiki/Forward_secrecy)\. Secrets Manager ensures encryption of your secret while in transit between AWS and the computers you use to retrieve the secret\.

### Automatically rotate your secrets<a name="features_autorotate"></a>

* Secrets Manager can rotate credentials automatically 
  * for
    * [*natively* supported AWS databases](#databases-with-fully-configured-and-ready-to-use-rotation-supporta-namefull-rotation-supporta)
      * *natively*  == NO additional programming
    * OTHER databases or services
      * requirements
        * create a custom Lambda function 
  * automatically == NO user intervention + scheduled
  * 👀rotation -- is done via an -- AWS Lambda function / perform the tasks: 👀
    + Creates a new version of the secret
    + Stores the secret | Secrets Manager
    + Configures the protected service -- to use the -- new version
    + Verifies the new version
    + Marks the new version -- as -- production ready

* Staging labels
  * uses
    * keep track of the different versions of your secrets 
  * rules
    * allowed >= 1 / version
    * each staging label -- can ONLY be attached to -- 1! version 
  * `AWSCURRENT`
    * 👀== label | currently active & in-use version of the secret 👀
    * uses
      * most common configured one -- to query -- your applications
  * `AWSPENDING`
    * 👀== label | new version of a secret 👀
      * UNTIL completed
        * testing
        * validation
      * ONCE it's completed -> pass to `AWSCURRENT`

#### Databases / fully configured & ready-to-use rotation support<a name="full-rotation-support"></a>

* Amazon RDS / AWS written + tested Lambda rotation function templates + full configuration of the rotation process
  + Amazon Aurora | Amazon RDS
  + MySQL | Amazon RDS
  + PostgreSQL | Amazon RDS
  + Oracle | Amazon RDS
  + MariaDB | Amazon RDS
  + Microsoft SQL Server | Amazon RDS

#### Other services / FULLY configured and ready-to-use rotation support<a name="other-with-full-rotation-support"></a>

* other FULLY supported 
  + Amazon DocumentDB 
  + Amazon Redshift 
* other databases or services
  * see [How rotation works](rotating-secrets.md#rotate-secrets_how) 

### Control access to secrets<a name="features_control-access"></a>

You can attach AWS Identity and Access Management \(IAM\) permission policies to your users, groups, and roles that grant or deny access to specific secrets, and restrict management of those secrets\. For example, you might attach one policy to a group with members that require the ability to fully manage and configure your secrets\. Another policy attached to a role used by an application might grant only read permission on the one secret the application needs to run\.

Alternatively, you can attach a resource\-based policy directly to the secret to grant permissions specifying users who can read or modify the secret and the versions\. Unlike an identity\-based policy which automatically applies to the user, group, or role, a resource\-based policy attached to a secret uses the `Principal` element to identify the target of the policy\. The `Principal` element can include users and roles from the same account as the secret or principals from other accounts\.

## Compliance with standards for AWS Secrets Manager<a name="asm_compliance"></a>

AWS Secrets Manager has undergone auditing for the following standards and can be part of your solution when you need to obtain compliance certification\.


|  |  | 
| --- |--- |
| ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/secretsmanager/latest/userguide/images/HIPAA.jpg) |  AWS has expanded its Health Insurance Portability and Accountability Act \(HIPAA\) compliance program to include AWS Secrets Manager as a [HIPAA\-eligible service](https://aws.amazon.com/compliance/hipaa-eligible-services-reference/)\. If you have an executed Business Associate Agreement \(BAA\) with AWS, you can use Secrets Manager to help build your HIPAA\-compliant applications\. AWS offers a [HIPAA\-focused whitepaper](https://docs.aws.amazon.com/whitepapers/latest/architecting-hipaa-security-and-compliance-on-aws/architecting-hipaa-security-and-compliance-on-aws.html) for customers who are interested in learning more about how they can leverage AWS for the processing and storage of health information\. For more information, see [HIPAA Compliance](https://aws.amazon.com/compliance/hipaa-compliance/)\.  | 
| ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/secretsmanager/latest/userguide/images/pci_aws.png) |  AWS Secrets Manager has an Attestation of Compliance for Payment Card Industry \(PCI\) Data Security Standard \(DSS\) version 3\.2 at Service Provider Level 1\. Customers who use AWS products and services to store, process, or transmit cardholder data can use AWS Secrets Manager as they manage their own PCI DSS compliance certification\. For more information about PCI DSS, including how to request a copy of the AWS PCI Compliance Package, see [PCI DSS Level 1](https://aws.amazon.com/compliance/pci-dss-level-1-faqs/)\.  | 
| ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/secretsmanager/latest/userguide/images/iso_aws.png) |  AWS Secrets Manager has successfully completed compliance certification for ISO/IEC 27001, ISO/IEC 27017, ISO/IEC 27018, and ISO 9001\. For more information, see [ISO 27001](https://aws.amazon.com/compliance/iso-27001-faqs/), [ISO 27017](https://aws.amazon.com/compliance/iso-27017-faqs/), [ISO 27018](https://aws.amazon.com/compliance/iso-27018-faqs/), [ISO 9001](https://aws.amazon.com/compliance/iso-9001-faqs/)\.  | 
| ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/secretsmanager/latest/userguide/images/soc_aws.png) |  System and Organization Control \(SOC\) reports are independent third\-party examination reports that demonstrate how Secrets Manager achieves key compliance controls and objectives\. The purpose of these reports is to help you and your auditors understand the AWS controls that are established to support operations and compliance\. For more information, see [SOC Compliance](https://aws.amazon.com/compliance/soc-faqs/)\.   | 
| ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/secretsmanager/latest/userguide/images/FedRAMP-PRIMARY-LOGO.png) | The Federal Risk and Authorization Management Program \(FedRAMP\) is a government\-wide program that provides a standardized approach to security assessment, authorization, and continuous monitoring for cloud products and services\. The FedRAMP Program also provides provisional authorizations for services and regions for East/West and GovCloud to consume government or regulated data\. For more information, see [ FedRAMP Compliance\.](https://aws.amazon.com/compliance/fedramp/) | 
| ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/secretsmanager/latest/userguide/images/DoD.jpg) | The Department of Defense \(DoD\) Cloud Computing Security Requirements Guide \(SRG\) provides a standardized assessment and authorization process for cloud service providers \(CSPs\) to gain a DoD provisional authorization, so that they can serve DoD customers\. For more information, see [ DoD SRG Resources](https://aws.amazon.com/compliance/dod/) | 
| ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/secretsmanager/latest/userguide/images/IRAP.jpg) | The Information Security Registered Assessors Program \(IRAP\) enables Australian government customers to validate that appropriate controls are in place and determine the appropriate responsibility model for addressing the requirements of the Australian government Information Security Manual \(ISM\) produced by the Australian Cyber Security Centre \(ACSC\)\. For more information, see [ IRAP Resources](https://aws.amazon.com/compliance/irap/) | 
| ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/secretsmanager/latest/userguide/images/compliance-privacy-singapore.png) | Amazon Web Services \(AWS\) achieved the Outsourced Service Provider’s Audit Report \(OSPAR\) attestation\. AWS alignment with the Association of Banks in Singapore \(ABS\) Guidelines on Control Objectives and Procedures for Outsourced Service Providers \(ABS Guidelines\) demonstrates to customers AWS commitment to meeting the high expectations for cloud service providers set by the financial services industry in Singapore\. For more information, see [ OSPAR Resources](https://aws.amazon.com/compliance/OSPAR/) | 

## Pricing for AWS Secrets Manager<a name="asm_pricing"></a>

* pay ONLY for what you use /
  * NO minimum or setup fees
  * if secretes marked for deletion -> NO

* see [AWS Secrets Manager Pricing](https://aws.amazon.com/secrets-manager/pricing)

* AWS managed key \(`aws/secretsmanager`\)
  * default built-in
  * used by 
    * Secrets Manager
  * pricing
    * encrypt for free
 
* created your own KMS keys
  * pricing
    * current AWS KMS rate

* AWS Lambda function
  * used | turn on automatic rotation
  * pricing
    * rotation function | current Lambda rate
  * see [AWS Lambda Pricing](http://aws.amazon.com/lambda/pricing/) 

* AWS CloudTrail
  * allows
    * obtain logs -- of the -- API calls / Secrets Manager sends out
      * logged -- as -- management events 
  * pricing
    * the first copy for free
    * depends on if you ALSO enable
      * Amazon S3 log storage
      * notification -- for -- Amazon SNS 
    * if you set up additional trails -> additional copies of management events -- can -- incur costs 
    * see [AWS CloudTrail pricing](https://aws.amazon.com/cloudtrail/pricing)

## Support and feedback for AWS Secrets Manager<a name="support-and-feedback"></a>

We welcome your feedback\. You can send comments to [awssecretsmanager\-feedback@amazon\.com](mailto:awssecretsmanager-feedback@amazon.com)\. You also can post your feedback and questions in our [AWS Secrets Manager support forum](https://forums.aws.amazon.com/forum.jspa?forumID=296)\. For more information about the AWS Support forums, see [Forums Help](https://forums.aws.amazon.com/help.jspa)\.

To request new features for the AWS Secrets Manager console or command line tools, we recommend you submit them in email to [awssecretsmanager\-feedback@amazon\.com](mailto:awssecretsmanager-feedback@amazon.com)\.

To provide feedback for our documentation, you can use the feedback link at the bottom of each web page\. Be specific about the issue you face and how the documentation failed to help you\. Let us know what you saw and how that differed from what you expected\. That helps us to understand what we need to do to improve the documentation\.

Here are some additional resources available to you:
+ **[AWS Training Catalog](https://aws.amazon.com/training/course-descriptions/)** – Role\-based and specialty courses, as well as self\-paced labs, to help you sharpen your AWS skills and gain practical experience\.
+ **[AWS Developer Tools](https://aws.amazon.com/developertools/)** – Tools and resources that provide documentation, code examples, release notes, and other information to help you build innovative applications with AWS\.
+ **[AWS Support Center](https://console.aws.amazon.com/support/home#/)** – The hub for creating and managing your AWS Support cases\. It includes links to other helpful resources, such as forums, technical FAQs, service health status, and AWS Trusted Advisor\.
+ **[AWS Support](https://aws.amazon.com/premiumsupport/)** – A one\-on\-one, fast\-response support channel for helping you build and run applications in the cloud\.
+ **[Contact Us](https://aws.amazon.com/contact-us/)** – A central contact point for inquiries about AWS billing, accounts, events, and other issues\.
+ **[AWS Site Terms](https://aws.amazon.com/terms/)** – Detailed information about our copyright and trademark, your account, your license, site access, and other topics\.