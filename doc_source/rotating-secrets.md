# Rotate AWS Secrets Manager secrets<a name="rotating-secrets"></a>

* see [gettingStarted](getting-started.md#rotationa-nameterm_rotationa) 

**Topics**
+ [How rotation works](#rotate-secrets_how)
+ [Managed rotation](rotate-secrets_managed.md)
+ [Automatic rotation for database secrets \(console\)](rotate-secrets_turn-on-for-db.md)
+ [Automatic rotation \(console\)](rotate-secrets_turn-on-for-other.md)
+ [Automatic rotation \(AWS CLI\)](rotate-secrets-cli.md)
+ [Rotate a secret immediately](rotate-secrets_now.md)
+ [Troubleshoot rotation](troubleshoot_rotation.md)

## How rotation works<a name="rotate-secrets_how"></a>

* how does it work?
  * 👀Secrets Manager rotation -- uses an -- AWS Lambda function 👀
    * see [AWS Lambda function Pricing](intro.md#asm_pricing)
    * -- according to the -- schedule set up
      * types os schedules
        * after a period of time
        * cron expression
      * see [Schedule expressions](rotate-secrets_schedule.md)
    * | rotation, Secrets Manager -- calls several times the -- SAME Lambda function / different parameters / time
      * JSON request structure 
        ```
        {
            "Step" : "request.type",
            "SecretId" : "string",
            "ClientRequestToken" : "string"
        }
        ```
  * if you ALSO manually update your secret value | automatic rotation is set up & AFTER calculating the next rotation date -> Secrets Manager considers the valid rotation

* [staging labels](https://docs.aws.amazon.com/secretsmanager/latest/userguide/getting-started.html#term_version)
  * allows
    * labeling secret versions | rotation

* rotation function
  * AWS Lambda function / makes the rotating
  * see [Logging AWS Secrets Manager events -- via -- AWS CloudTrail](retrieve-ct-entries.md)
  * 👀if it's | Amazon RDS (!= Oracle\) & Amazon DocumentDB -> -- to connect to your database, automatically use  -- 👀
    * if it's available (see [Determine when your rotation function was created](troubleshoot_rotation.md#rotation-function-created-date))
      * SSL, OR
      * TLS
    * otherwise
      * unencrypted connection
  * == 4 steps
    1. `createSecret`
       1. == create a new version of the secret / 
          1. can contain
             1. NEW password,
             2. NEW username & password
             3. MORE secret information
          2. staging label `AWSPENDING`
    2. `setSecret`
       1. allows
          1. changing the credentials | database or service / == credentials | `AWSPENDING` version of the secret
          2. if [rotation strategy is alternate users](getting-started.md#rotation-strategy-alternating-usersa-namerotating-secrets-two-usersa) -> create a new user / 's permissions == existing user's permissions
    3. `testSecret`
       1. == test the new secret version (`AWSPENDING`)
       2. -- based on the -- type of access / your applications need
          1. if rotation functions -- based on -- [Rotation function templates](reference_available-rotation-templates.md) -> new secret is tested -- via -- read access
          2. update the function / include OTHER access (_Example:_ write access)
    4. `finishSecret`
       1. == finish the rotation
       2. what happens?
          1. the label `AWSCURRENT` | previous secret version -- is moved to -- this version
          2. `AWSPREVIOUS` staging label is added | previous version
             1. == you retain the last known good version of the secret 
  * if ANY rotation step fails -> Secrets Manager retries the ENTIRE rotation process / MULTIPLE times
  * if rotation is 
    * successful -> `AWSPENDING` staging label -- might 
      * be attached to the -- == `AWSCURRENT` version 
      * NOT be attached to --
        * ANY version
        * == `AWSCURRENT` version -> ANY later invocation of rotation 
          * -- assumes that --  previous rotation request STILL in progress
          * returns an error
    * unsuccessful -> `AWSPENDING` staging label -- might be attached to an -- empty secret version
      * see [Troubleshoot rotation](troubleshoot_rotation.md)

* TODO:
After rotation is successful, applications that [Retrieve secrets from AWS Secrets Manager](retrieving-secrets.md) from Secrets Manager automatically get the updated credentials\. For more details about how each step of rotation works, see the [AWS Secrets Manager rotation function templates](reference_available-rotation-templates.md)\.