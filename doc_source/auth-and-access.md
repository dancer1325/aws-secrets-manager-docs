# Authentication and access control for AWS Secrets Manager<a name="auth-and-access"></a>

* [AWS IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)
  * allows
    * 👀securing access to Secrets Manager's secrets 👀
  * provides
    * authentication
      * verifies the identity of individuals' requests -- via --
        * sign-in process with
          * passwords,
          * access keys,
          * MFA tokens
        * see [Signing in to AWS](https://docs.aws.amazon.com/IAM/latest/UserGuide/console.html)
    * access control
      * ensures that ONLY approved individuals -- can perform -- operations | AWS resources (_Example:_ secrets) /
        * defined by policies
          * see [Policies and permissions in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html)

## Secrets Manager administrator permissions<a name="auth-and-access_admin"></a>

* follow [Adding and removing IAM identity permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html)
* attach the following policies
  + [https://docs.aws.amazon.com/secretsmanager/latest/userguide/reference_available-policies.html](https://docs.aws.amazon.com/secretsmanager/latest/userguide/reference_available-policies.html)
  + [https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html#aws-managed-policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html#aws-managed-policies)

* recommendations
  * NOT grant administrator permissions | end users
    * Reason: 🧠`IAMFullAccess` permission required to enable rotation -> grants significant permissions 🧠

## Permissions to access secrets<a name="auth-and-access_secrets"></a>

* *IAM permissions policy*
  * describes 
    * who (user or services) can perform
    * which actions
    * | which resources (your secrets)
  * you can 
    + [Attach a permissions policy | identity](auth-and-access_iam-policies.md)
    + [Attach a permissions policy | AWS Secrets Manager secret](auth-and-access_resource-policies.md)

## Permissions for Lambda rotation functions<a name="auth-and-access_rotate"></a>

* AWS Lambda functions / [rotate secrets](https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotating-secrets.html)
  * requirements
    * MUST have access to the
      * secret
      * database or service 
  * see [Permissions for rotation](rotating-secrets-required-permissions-function.md)

## Permissions for encryption keys<a name="auth-and-access_encrypt"></a>

* AWS KMS keys / [encrypt secrets](https://docs.aws.amazon.com/secretsmanager/latest/userguide/security-encryption.html)
  * requirements
    * if you use 
      * the AWS managed key `aws/secretsmanager` -> has the correct permissions
      * 👀another AWS KMS key -> Secrets Manager needs permissions to that key 👀
        * see [Permissions for the KMS key](security-encryption.md#security-encryption-authz) 