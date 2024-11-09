# Managed rotation for AWS Secrets Manager secrets<a name="rotate-secrets_managed"></a>

* 👀== service configures & manages rotation -- for -- you 👀
  * -> NOT AWS Lambda function used
  * _Example:_
    + Amazon RDS
      + managed rotation -- for -- master user credentials
      + see [Password management with Amazon RDS and AWS Secrets Manager](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-secrets-manager.html) 
    + Aurora
      + managed rotation -- for -- master user credentials
      + see [Password management with Amazon Aurora and AWS Secrets Manager](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/rds-secrets-manager.html) 
* **how to change the schedule for managed rotation?**
  * | AWS Console
    1. Open the managed secret | Secrets Manager console -- via --
       1. follow a link | managing service, or 
       2. [search for the secret](service-linked-secrets.md) | Secrets Manager console
    2. | **Rotation schedule**
       1. enter your schedule in UTC time zone | **Schedule expression builder** or as a **Schedule expression**
          1. == -- via -- `rate()` or `cron()` expression 
          2. rotation window automatically starts | midnight -- unless you specify a **Start time** -- 
          3. see [Schedule expressions](rotate-secrets_schedule.md)
    3. **Window duration**
       1. optional
       2. == length of the window | which you want Secrets Manager to rotate your secret
          1. _Example:_ **3h** == 3 hour window\
          2. recommendations
             1. < NEXT rotation window
          3. if rotation schedule in
             1. hours -> window automatically closes AFTER 1 hour
             2. days -> the window automatically closes | end of the day
  * | AWS CLI
    + call [rotate-secret](https://docs.aws.amazon.com/cli/latest/reference/secretsmanager/rotate-secret.html)
      + see [Schedule expressions](rotate-secrets_schedule.md)
      + _Example:_ rotates the secret between 16:00 and 18:00 UTC | 1st and 15th day of the month 

        ```
        aws secretsmanager rotate-secret \
            --secret-id MySecret \
            --rotation-rules "{\"ScheduleExpression\": \"cron(0 16 1,15 * ? *)\", \"Duration\": \"2h\"}"
        ```