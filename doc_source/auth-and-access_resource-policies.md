# Attach a permissions policy | AWS Secrets Manager secret<a name="auth-and-access_resource-policies"></a>

* IAM resource-based policy
  * uses
    + grant access
      + | 1! secret -- to -- multiple users or roles 
      + -- to -- users or roles | other AWS accounts
  + ways to attach | secret -- via --
    + AWS Console
      + Secrets Manager -- uses -- [Zelkova](https://aws.amazon.com/blogs/security/protect-sensitive-data-in-the-cloud-with-automated-reasoning-zelkova/) & `ValidateResourcePolicy` API 
        + [Zelkova](https://aws.amazon.com/blogs/security/protect-sensitive-data-in-the-cloud-with-automated-reasoning-zelkova/)
          + == automated reasoning engine
        + Reason: 🧠 prevent you from granting a wide range of IAM principals access to your secrets 🧠
    + AWS CLI or SDK
      + call the `PutResourcePolicy` API / `BlockPublicPolicy` parameter
  * see [permissions policy examples -- for -- AWS Secrets Manager](auth-and-access_examples.md)
  * steps to view, change, or delete the resource policy -- for a -- Secret Manager's secret | console
    1. open the Secrets Manager console
    2. | **Resource permissions** section, choose **Edit permissions**
    3. | code field
       + attach or modify a resource policy / enter the policy 
       + delete the policy, clear the code field

## AWS CLI<a name="auth-and-access_resource_cli"></a>

* `aws secretsmanager`
  * `get-resource-policy` 
    * retrieves the resource-based policy / attached | secret
    * see [get-resource-policy](https://docs.aws.amazon.com/cli/latest/reference/secretsmanager/get-resource-policy.html)
    * _Example:_ 

        ```
        aws secretsmanager get-resource-policy \
            --secret-id MyTestSecret
        ```
  * `delete-resource-policy`
    * deletes the resource-based policy / attached | secret
    * see [delete-resource-policy.html](https://docs.aws.amazon.com/cli/latest/reference/secretsmanager/delete-resource-policy.html)
    * _Example:_ 

      ```
      aws secretsmanager delete-resource-policy \
        --secret-id MyTestSecret
      ```

  * `put-resource-policy`
    * adds a permissions policy | a secret  
      * / check first that the policy -- does NOT provide broad access to the -- secret 
    * see [put-resource-policy.html](https://docs.aws.amazon.com/cli/latest/reference/secretsmanager/put-resource-policy.html)  
    * _Example:_

      ```
      aws secretsmanager put-resource-policy \
        --secret-id MyTestSecret \
        --resource-policy file://mypolicy.json \
        --block-public-policy
      ```
    
      "mypolicy.json"  

        ```
        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "AWS": "arn:aws:iam::123456789012:role/MyRole"
                    },
                    "Action": "secretsmanager:GetSecretValue",
                    "Resource": "*"
                }
            ]
        }
        ```

## AWS SDK<a name="auth-and-access_resource_sdk"></a>

* [GetResourcePolicy.html](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_GetResourcePolicy.html)
  * retrieves the resource-based policy / attached | secret
* [DeleteResourcePolicy.html](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_DeleteResourcePolicy.html)
  * deletes the resource-based policy / attached | secret
* [PutResourcePolicy.html](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_PutResourcePolicy.html)
  * adds a permissions policy | a secret
    * / check first that the policy -- does NOT provide broad access to the -- secret
    * if there is ALREADY a policy attached -> the command
      * removes the existing
      * attaches the new policy\
    * policy -- must be formatted as -- JSON structured text
      * see [JSON policy document structure](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html#policies-introduction)
