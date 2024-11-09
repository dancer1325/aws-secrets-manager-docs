# Get started with AWS Secrets Manager<a name="getting-started"></a>

* 👀types of secrets / can be stored | AWS 👀
  + AWS credentials
    + [AWS Identity and Access Management](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)
  + Encryption keys
    + [AWS Key Management Service](https://docs.aws.amazon.com/kms/latest/developerguide/overview.html)
  + SSH keys
    + [Amazon EC2 Instance Connect](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Connect-using-EC2-Instance-Connect.html)
  + Private keys and certificates
    + [AWS Certificate Manager](https://docs.aws.amazon.com/acm/latest/userguide/acm-overview.html)
  + **Database credentials**
    + AWS Secrets Manager
  + **Application credentials**
    + AWS Secrets Manager
  + **OAuth tokens**
    + AWS Secrets Manager
  + **API**
    + AWS Secrets Manager

## Secrets Manager concepts<a name="getting-started_concepts"></a>

### Secret<a name="term_secret"></a>

* := secret information
* 👀== *secret value* + metadata about the secret 👀
  * secret value
    * allowed types
      * string
      * binary
    * recommendations
      * if you want to store MULTIPLE string values | 1 secret -> use a JSON text string / key/value pairs
        * _Example:_

            ```
            {
              "host"       : "ProdServer-01.databases.example.com",
              "port"       : "8888",
              "username"   : "administrator",
              "password"   : "EXAMPLE-PASSWORD",
              "dbname"     : "MyDatabase",
              "engine"     : "mysql"
            }
            ```
  * metadata ==
    + information as
      + secretName,
      + description,
      + resource policy,
      + tags
      + rotation of the secret
        + see [Rotation](#rotationa-nameterm_rotationa) 
    + ARN -- for an -- *encryption key*
      + == AWS KMS key / Secrets Manager -- uses to --
        + encrypt the secret value
        + decrypt the secret value
      + Secrets Manager 
        + stores secret text | encrypted form
        + encrypts the secret | in transit
      + see [Secret encryption and decryption in AWS Secrets Manager](security-encryption.md)
      + format
  
        ```
        arn:aws:secretsmanager:<Region>:<AccountId>:secret:SecretName-6RandomCharacters
        ```

* Secrets Manager -- via, IAM permission policies, -- guarantee that ONLY authorized users can
  * access a secret OR
  * modify a secret
  * see [Authentication and access control for AWS Secrets Manager](auth-and-access.md)

* *versions* of the secret
  * hold copies of the encrypted secret value
  * when does Secrets Manager creates a new version?
    * change the secret value
    * secret is rotated 
  * see [Version](#term_version)

* *replicating* a secret
  * allows
    * using a secret / ACROSS MULTIPLE AWS Regions
  * *replica secret*
    * == copy of the original OR *primary secret* /
      * -- linked to the -- primary secret
  * see [Replicate an AWS Secrets Manager secret to other AWS Regions](create-manage-multi-region-secrets.md)

* see [Create and manage secrets with AWS Secrets Manager](managing-secrets.md)

### Rotation<a name="term_rotation"></a>

* := process of periodically updating a secret 
  * | 
    * secret &
    * database OR service
  * Reason: 🧠 make it more difficult -- for an attacker, to access the -- credentials
  * types
    * automatic rotation
    * managed rotation
      * use cases
        * [Secrets / -- managed by -- OTHER services](service-linked-secrets.md) 
      * see [Managed rotation](rotate-secrets_managed.md)
      * requirements
        * create the secret -- through the -- managing service
* see [Rotate AWS Secrets Manager secrets](rotating-secrets.md)

### Rotation strategy<a name="rotation-strategy"></a>

#### Rotation strategy: 1! user<a name="rotating-secrets-one-user-one-password"></a>

* updates credentials / for 1 user | 1 secret
  * requirements
    * user MUST have permission -- to -- update their password 
* simplest rotation strategy
* use cases
  * MOST of them
* how does it work?
  * | secret rotation
    * open database connections are NOT dropped
    * 👀 short period of time between [password | database changes, secret is updated] 👀
      * -> side effect
        * database deny calls / use the rotated credentials
        * 💡solution💡
          * [appropriate retry strategy](http://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/)
  * | after rotation
    * new connections -- use the -- NEW credentials 

#### Rotation strategy: alternating users<a name="rotating-secrets-two-users"></a>

* updates credentials / for 2 users | 1 secret
  * requirements
    * create the first user
    * provide the credentials -- for a -- `superuser` | ANOTHER secret
      * Reason: 🧠 MOST users do NOT have permission -- to -- clone themselves 🧠
* hoes does it work?
  * | first rotation
    * 👀rotation function -- clones -- the user == create the second user 👀
      * recommendations
        * if cloned users is *ad hoc* or *interactive* user (== cloned users' permissions != original user's permissions) -> use single-user rotation strategy
  * 👀 | next rotations
    * rotation function -- alternates -- which user's password it updates 👀
  * if an application retrieves the secret | rotation -> STILL valid set of credentials
  * | AFTER rotation, 
    * `user` credential is valid
    * `user_clone` credential is valid 
* uses 
  * databases' permission models /
    * first role -- owns the -- database tables
    * second role's permission -- can access the -- database tables 
  * applications / require high availability 
* vs 1! user rotation
  * 👀LESS chance of applications / get a deny  👀
* use cases 
  * if the database is hosted | server farm / password change takes time -- to propagate to -- ALL servers -> chance database deny calls / use the rotated credentials
    * Solution: 💡[appropriate retry strategy](http://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/) 💡 

### Version<a name="term_version"></a>

* TODO:
A secret has *versions* which hold copies of the encrypted secret value\. When you change the secret value, or the secret is rotated, Secrets Manager creates a new version\.

Secrets Manager doesn't store a history of secrets with versions\. Instead, it keeps track of the previous secret value by tagging that secret version with `AWSPREVIOUS`\. During rotation, Secrets Manager also tracks the next secret value by tagging that secret version with `AWSPENDING`\. A secret always has an `AWSCURRENT` version\. The `AWSCURRENT` version is what Secrets Manager returns when you retrieve the secret value, unless you specify a different version\. 

You can add other versions with your own labels by using [https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_UpdateSecretVersionStage.html](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_UpdateSecretVersionStage.html) in the AWS CLI or an AWS SDK\. You can attach up to 20 staging labels to a secret\. Two versions of a secret can't have the same staging label\. If a version doesn't have a label, Secrets Manager considers it deprecated\. Secrets Manager removes deprecated secret versions when there are more than 100\. Secrets Manager doesn't remove versions created less than 24 hours ago\.