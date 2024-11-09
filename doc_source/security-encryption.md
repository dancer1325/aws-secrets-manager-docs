# Secret encryption and decryption | AWS Secrets Manager<a name="security-encryption"></a>

* [envelope encryption](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#enveloping)
  * uses
    * 👀protect EACH Secret Manager secret value 👀
  * use  cases
    * if secret's value changes -> Secrets Manager -- generates a -- NEW data key
  * data key
    * how is it generated?
      * 👀KMS key -- is encrypted, via -- 256-bit AES symmetric 👀 
    * stored | secret's metadata
    * plaintext 

* how to decrypt the secret?
  * Secrets Manager -- decrypts, via KMS key | AWS KMS, the -- encrypted data key  

* encrypted secret value
  * stored | outside of AWS KMS
  * 's metadata
    * encrypted copy of the data key

* | create a secret
  * -> you can choose 
    * ANY symmetric encryption customer managed key | AWS account & Region
      * BUT ONLY symmetric is supported
    * AWS managed key for Secrets Manager \(`aws/secretsmanager`\)\
      * == default 
    * about KMS key
      * the same
      * different KMS keys / EACH secret | your account 
        * uses
          * set custom permissions | keys / group of secrets, or
          * audit particular operations / those keys
  * if you use a KMS key | [external key store](https://docs.aws.amazon.com/kms/latest/developerguide/keystore-external.html) -> cryptographic operations | KMS key -- might
    * take longer
    * be less
      * reliable
        * Reason: 🧠 the request has to travel outside of AWS 🧠
      * durable 

* change the secret's encryption key
  * ways
    * | AWS console
    * AWS CLI
    * AWS SDK
      * -- via -- [UpdateSecret](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_UpdateSecret.html)
  * -> Secrets Manager
    * re-encrypts (| new encryption key) versions of the secret / have the staging labels `AWSCURRENT`, `AWSPENDING`, and `AWSPREVIOUS` 
    * encrypts it -- via the -- new key
  * how to decrypt the secret?
    * -- via --
      * old key
      * new key

* ways to find the KMS key -- associated with a -- secret
  * view the secret | AWS Console or
  * call
    * [ListSecrets](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_ListSecrets.html) or
    * [DescribeSecret](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_DescribeSecret.html)
    * if the secret -- is associated with the -- AWS managed key for Secrets Manager \(`aws/secretsmanager`\) -> p operations do NOT return a KMS key identifier

**Topics**
+ [Encryption and decryption processes](#security-encryption-encrypt)
+ [How Secrets Manager uses your KMS key](#security-encryption-using-cmk)
+ [Permissions for the KMS key](#security-encryption-authz)
+ [Secrets Manager encryption context](#security-encryption-encryption-context)
+ [Monitor Secrets Manager interaction with AWS KMS](#security-encryption-logs)

## Encryption and decryption processes<a name="security-encryption-encrypt"></a>

* Secrets Manager process to encrypt the secret value
  1. Secrets Manager -- calls the -- AWS KMS [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) operation /
     1. inputs 
        1. KMS key's ID
        2. 256\-bit AES symmetric key 
     2. AWS KMS returns a
        1. plaintext data key
        2. copy of the encrypted data key
  2. once it's generated -> Secrets Manager removes the plaintext key from memory
  3. Secrets Manager stores the encrypted data key | secret's metadata
     1. allows
        1. decrypting the secret value
     2. 👀NONE of the Secrets Manager APIs return the 👀
        1. encrypted secret or
        2. encrypted data key

* Secrets Manager process to decrypt an encrypted secret value
  1. Secrets Manager -- calls the -- AWS KMS [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation /
     1. inputs
        1. encrypted data key
  2. AWS KMS returns the plaintext data key
  3. Secrets Manager
     1. -- decrypts, via the plaintext data key, -- the secret value
     2. -- removes the -- data key from memory

## How Secrets Manager uses your KMS key<a name="security-encryption-using-cmk"></a>

* TODO:
Secrets Manager uses the KMS key that is associated with a secret to generate a data key for each secret value\. Secrets Manager also uses the KMS key to decrypt that data key when it needs to decrypt the encrypted secret value\. You can track the requests and responses in AWS CloudTrail events, [Amazon CloudWatch Logs](https://docs.aws.amazon.com/kms/latest/developerguide/services-secrets-manager.html#asm-logs), and audit trails\.

The following Secrets Manager operations trigger a request to use your KMS key\.

**GenerateDataKey**  
Secrets Manager calls the AWS KMS [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) operation in response to the following Secrets Manager operations\.  
+ [CreateSecret](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_CreateSecret.html) – If the new secret includes a secret value, Secrets Manager requests a new data key to encrypt it\. 
+ [PutSecretValue](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_PutSecretValue.html)– Secrets Manager requests a new data key to encrypt the specified secret value\.
+ [UpdateSecret](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_UpdateSecret.html) – If you change the secret value or the KMS key, Secrets Manager requests a new data key to encrypt the new secret value\.
The [RotateSecret](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_RotateSecret.html) operation does not call `GenerateDataKey`, because it does not change the secret value\. However, if the Lambda function that `RotateSecret` invokes changes the secret value, its call to the `PutSecretValue` operation triggers a `GenerateDataKey` request\.

**Decrypt**  
To decrypt an encrypted secret value, Secrets Manager calls the AWS KMS [Decrypt](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_Decrypt.html) operation to decrypt the encrypted data key in the secret\. Then, it uses the plaintext data key to decrypt the encrypted secret value\.  
Secrets Manager calls the [Decrypt](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_Decrypt.html) operation in response to the following Secrets Manager operations\.   
+ [GetSecretValue](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_GetSecretValue.html) – Secrets Manager decrypts the secret value before returning it to the caller\.
+ [PutSecretValue](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_PutSecretValue.html) and [UpdateSecret](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_UpdateSecret.html) – Most `PutSecretValue` and `UpdateSecret` requests do not trigger a `Decrypt` operation\. However, when a `PutSecretValue` or `UpdateSecret` request attempts to change the secret value in an existing version of a secret, Secrets Manager decrypts the existing secret value and compares it to the secret value in the request to confirm that they are the same\. This action ensures the that Secrets Manager operations are idempotent\.

**Validating access to the KMS key**  
When you establish or change the KMS key that is associated with secret, Secrets Manager calls the `GenerateDataKey` and `Decrypt` operations with the specified KMS key\. These calls confirm that the caller has permission to use the KMS key for these operation\. Secrets Manager discards the results of these operations; it does not use them in any cryptographic operation\.  
You can identify these validation calls because the value of the `SecretVersionId` key [encryption context](https://docs.aws.amazon.com/kms/latest/developerguide/services-secrets-manager.html#asm-encryption-context) in these requests is `RequestToValidateKeyAccess`\.  
In the past, Secrets Manager validation calls did not include an encryption context\. You might find calls with no encryption context in older AWS CloudTrail logs\.

## Permissions for the KMS key<a name="security-encryption-authz"></a>

When Secrets Manager uses a KMS key in cryptographic operations, it acts on behalf of the user who is accessing or updating the secret value\.
+ To retrieve a secret, the user must have `kms:Decrypt` permission granted in either an IAM policy or a key policy\.
+ To update the secret, the user must have both `kms:Decrypt` and `kms:GenerateDataKey` permission granted in either an IAM policy or a key policy\.

To allow the KMS key to be used only for requests that originate in Secrets Manager, you can use the [kms:ViaService condition key](https://docs.aws.amazon.com/kms/latest/developerguide/policy-conditions.html#conditions-kms-via-service) with the `secretsmanager.<Region>.amazonaws.com` value\.

You can also use the keys or values in the [encryption context](https://docs.aws.amazon.com/kms/latest/developerguide/services-secrets-manager.html#asm-encryption-context) as a condition for using the KMS key for cryptographic operations\. For example, you can use a [string condition operator](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html#Conditions_String) in an IAM or key policy document, or use a [grant constraint](https://docs.aws.amazon.com/kms/latest/APIReference/API_GrantConstraints.html) in a grant\.

### Key policy of the AWS managed key \(`aws/secretsmanager`\)<a name="security-encryption-policies"></a>

The key policy for the AWS managed key for Secrets Manager \(`aws/secretsmanager`\) gives users permission to use the KMS key for specified operations only when Secrets Manager makes the request on the user's behalf\. The key policy does not allow any user to use the KMS key directly\.

This key policy, like the policies of all [AWS managed keys](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys), is established by the service\. You cannot change the key policy, but you can view it at any time\. For details, see [ Viewing a key policy](https://docs.aws.amazon.com/kms/latest/developerguide/key-policy-viewing.html)\.

The policy statements in the key policy have the following effect:
+ Allow users in the account to use the KMS key for cryptographic operations only when the request comes from Secrets Manager on their behalf\. The `kms:ViaService` condition key enforces this restriction\.
+ Allows the AWS account to create IAM policies that allow users to view KMS key properties and revoke grants\.
+ Although Secrets Manager does not use grants to gain access to the KMS key, the policy also allows Secrets Manager to [create grants](https://docs.aws.amazon.com/kms/latest/developerguide/grants.html) for the KMS key on the user's behalf and allows the account to [revoke any grant](https://docs.aws.amazon.com/kms/latest/APIReference/API_RevokeGrant.html) that allows Secrets Manager to use the KMS key\. These are standard elements of policy document for an AWS managed key\.

The following is a key policy for an example AWS managed key for Secrets Manager\.

```
{
  "Version" : "2012-10-17",
  "Id" : "auto-secretsmanager-1",
  "Statement" : [ {
    "Sid" : "Allow access through AWS Secrets Manager for all principals in the account that are authorized to use AWS S
ecrets Manager",
    "Effect" : "Allow",
    "Principal" : {
      "AWS" : "*"
    },
    "Action" : [ "kms:Decrypt", "kms:ReEncrypt*", "kms:GenerateDataKey*", "kms:CreateGrant", "kms:Describ
eKey" ],
    "Resource" : "*",
    "Condition" : {
      "StringEquals" : {
        "kms:ViaService" : "secretsmanager.us-west-2.amazonaws.com",
        "kms:CallerAccount" : "111122223333"
      }
    }
  },{
    "Sid" : "Allow direct access to key metadata to the account",
    "Effect" : "Allow",
    "Principal" : {
      "AWS" : "arn:aws:iam::111122223333:root"
    },
    "Action" : [ "kms:Describe*", "kms:Get*", "kms:List*", "kms:RevokeGrant" ],
    "Resource" : "*"
  } ]
}
```

## Secrets Manager encryption context<a name="security-encryption-encryption-context"></a>

An [encryption context](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#encrypt_context) is a set of key–value pairs that contain arbitrary nonsecret data\. When you include an encryption context in a request to encrypt data, AWS KMS cryptographically binds the encryption context to the encrypted data\. To decrypt the data, you must pass in the same encryption context\. 

In its [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) and [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) requests to AWS KMS, Secrets Manager uses an encryption context with two name–value pairs that identify the secret and its version, as shown in the following example\. The names do not vary, but combined encryption context values will be different for each secret value\.

```
"encryptionContext": {
    "SecretARN": "arn:aws:secretsmanager:us-east-2:111122223333:secret:test-secret-a1b2c3",
    "SecretVersionId": "EXAMPLE1-90ab-cdef-fedc-ba987SECRET1"
}
```

You can use the encryption context to identify these cryptographic operation in audit records and logs, such as [AWS CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html) and Amazon CloudWatch Logs, and as a condition for authorization in policies and grants\.

The Secrets Manager encryption context consists of two name\-value pairs\.
+ **SecretARN** – The first name–value pair identifies the secret\. The key is `SecretARN`\. The value is the Amazon Resource Name \(ARN\) of the secret\.

  ```
  "SecretARN": "ARN of an Secrets Manager secret"
  ```

  For example, if the ARN of the secret is `arn:aws:secretsmanager:us-east-2:111122223333:secret:test-secret-a1b2c3`, the encryption context would include the following pair\.

  ```
  "SecretARN": "arn:aws:secretsmanager:us-east-2:111122223333:secret:test-secret-a1b2c3"
  ```
+ **SecretVersionId** – The second name–value pair identifies the version of the secret\. The key is `SecretVersionId`\. The value is the version ID\.

  ```
  "SecretVersionId": "<version-id>"
  ```

  For example, if the version ID of the secret is `EXAMPLE1-90ab-cdef-fedc-ba987SECRET1`, the encryption context would include the following pair\.

  ```
  "SecretVersionId": "EXAMPLE1-90ab-cdef-fedc-ba987SECRET1"
  ```

When you establish or change the KMS key for a secret, Secrets Manager sends [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) and [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) requests to AWS KMS to validate that the caller has permission to use the KMS key for these operations\. It discards the responses; it does not use them on the secret value\.

In these validation requests, the value of the `SecretARN` is the actual ARN of the secret, but the `SecretVersionId` value is `RequestToValidateKeyAccess`, as shown in the following example encryption context\. This special value helps you to identify validation requests in logs and audit trails\.

```
"encryptionContext": {
    "SecretARN": "arn:aws:secretsmanager:us-east-2:111122223333:secret:test-secret-a1b2c3",
    "SecretVersionId": "RequestToValidateKeyAccess"
}
```

**Note**  
In the past, Secrets Manager validation requests did not include an encryption context\. You might find calls with no encryption context in older AWS CloudTrail logs\.

## Monitor Secrets Manager interaction with AWS KMS<a name="security-encryption-logs"></a>

You can use AWS CloudTrail and Amazon CloudWatch Logs to track the requests that Secrets Manager sends to AWS KMS on your behalf\. For information about monitoring the use of secrets, see [Monitor AWS Secrets Manager secrets](monitoring.md)\.

**GenerateDataKey**  
When you [create or change](https://docs.aws.amazon.com/kms/latest/developerguide/services-secrets-manager.html#asm-using-cmk) the secret value in a secret, Secrets Manager sends a *[GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html)* request to AWS KMS that specifies the KMS key for the secret\.   
The event that records the `GenerateDataKey` operation is similar to the following example event\. The request is invoked by `secretsmanager.amazonaws.com`\. The parameters include the Amazon Resource Name \(ARN\) of the KMS key for the secret, a key specifier that requires a 256\-bit key, and the [encryption context](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#encrypt_context) that identifies the secret and version\.  

```
{
    "eventVersion": "1.05",
    "userIdentity": {
        "type": "IAMUser",
        "principalId": "AROAIGDTESTANDEXAMPLE:user01",
        "arn": "arn:aws:sts::111122223333:assumed-role/Admin/user01",
        "accountId": "111122223333",
        "accessKeyId": "AKIAIOSFODNN7EXAMPLE",
        "sessionContext": {
            "attributes": {
                "mfaAuthenticated": "false",
                "creationDate": "2018-05-31T23:23:41Z"
            }
        },
        "invokedBy": "secretsmanager.amazonaws.com"
    },
    "eventTime": "2018-05-31T23:23:41Z",
    "eventSource": "kms.amazonaws.com",
    "eventName": "GenerateDataKey",
    "awsRegion": "us-east-2",
    "sourceIPAddress": "secretsmanager.amazonaws.com",
    "userAgent": "secretsmanager.amazonaws.com",
    "requestParameters": {
        "keyId": "arn:aws:kms:us-east-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
        "keySpec": "AES_256",
        "encryptionContext": {
            "SecretARN": "arn:aws:secretsmanager:us-east-2:111122223333:secret:test-secret-a1b2c3",
            "SecretVersionId": "EXAMPLE1-90ab-cdef-fedc-ba987SECRET1"
        }
    },
    "responseElements": null,
    "requestID": "a7d4dd6f-6529-11e8-9881-67744a270888",
    "eventID": "af7476b6-62d7-42c2-bc02-5ce86c21ed36",
    "readOnly": true,
    "resources": [
        {
            "ARN": "arn:aws:kms:us-east-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
            "accountId": "111122223333",
            "type": "AWS::KMS::Key"
        }
    ],
    "eventType": "AwsApiCall",
    "recipientAccountId": "111122223333"
}
```

**Decrypt**  
Whenever you [get or change](https://docs.aws.amazon.com/kms/latest/developerguide/services-secrets-manager.html#asm-using-cmk) the secret value of a secret, Secrets Manager sends a [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) request to AWS KMS to decrypt the encrypted data key\.  
The event that records the `Decrypt` operation is similar to the following example event\. The user is the principal in your AWS account who is accessing the table\. The parameters include the encrypted table key \(as a ciphertext blob\) and the [encryption context](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#encrypt_context) that identifies the table and the AWS account\. AWS KMS derives the ID of the KMS key from the ciphertext\.   

```
{
    "eventVersion": "1.05",
    "userIdentity": {
        "type": "IAMUser",
        "principalId": "AROAIGDTESTANDEXAMPLE:user01",
        "arn": "arn:aws:sts::111122223333:assumed-role/Admin/user01",
        "accountId": "111122223333",
        "accessKeyId": "AKIAIOSFODNN7EXAMPLE",
        "sessionContext": {
            "attributes": {
                "mfaAuthenticated": "false",
                "creationDate": "2018-05-31T23:36:09Z"
            }
        },
        "invokedBy": "secretsmanager.amazonaws.com"
    },
    "eventTime": "2018-05-31T23:36:09Z",
    "eventSource": "kms.amazonaws.com",
    "eventName": "Decrypt",
    "awsRegion": "us-east-2",
    "sourceIPAddress": "secretsmanager.amazonaws.com",
    "userAgent": "secretsmanager.amazonaws.com",
    "requestParameters": {
        "encryptionContext": {
            "SecretARN": "arn:aws:secretsmanager:us-east-2:111122223333:secret:test-secret-a1b2c3",
            "SecretVersionId": "EXAMPLE1-90ab-cdef-fedc-ba987SECRET1"
        }
    },
    "responseElements": null,
    "requestID": "658c6a08-652b-11e8-a6d4-ffee2046048a",
    "eventID": "f333ec5c-7fc1-46b1-b985-cbda13719611",
    "readOnly": true,
    "resources": [
        {
            "ARN": "arn:aws:kms:us-east-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
            "accountId": "111122223333",
            "type": "AWS::KMS::Key"
        }
    ],
    "eventType": "AwsApiCall",
    "recipientAccountId": "111122223333"
}
```