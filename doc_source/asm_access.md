# Access AWS Secrets Manager<a name="asm_access"></a>

**Topics**
+ [Secrets Manager console](#asm-console)
+ [Command line tools](#asm-cli)
+ [AWS SDKs](#asm-sdks)
+ [HTTPS Query API](#asm-sdks_query-api)
+ [Secrets Manager endpoints](#endpoints)

## Secrets Manager console<a name="asm-console"></a>

* browser-based [Secrets Manager console](https://console.aws.amazon.com/secretsmanager/)
  * allows
    * performing ALMOST ANY task / -- related to -- your secrets

## AWS CL tools<a name="asm-cli"></a>

* vs console
  * faster
  * more convenient
* uses
  * build scripts / perform AWS tasks
* risks
  * command history being accessed
  * utilities -- have access to -- your command parameters
  * [how to mitigate them](security_cli-exposure-risks.md)

## AWS SDKs<a name="asm-sdks"></a>

* AWS SDKs
  * == libraries + sample code + tasks
    * _Example of tasks:_
      * cryptographically signing requests
      * managing errors,
      * retrying requests automatically 
  * available | SEVERAL programming languages & platforms
    * [Java](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/secretsmanager/package-summary.html),
    * [Python](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/secretsmanager.html),
    * [Ruby](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/SecretsManager.html),
    * [.NET](https://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/SecretsManager/NSecretsManagerModel.html),
    * [Node\.js](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/SecretsManager.html)
    * [Go](https://docs.aws.amazon.com/sdk-for-go/api/service/secretsmanager/)
    * [PHP](https://docs.aws.amazon.com//aws-sdk-php/v3/api/namespace-Aws.SecretsManager.html)
    * [C\+\+](http://sdk.amazonaws.com/cpp/api/LATEST/namespace_aws_1_1_secrets_manager.html)
  * see 
    * [AWS SDKs](#asm-sdks)
    * [Tools for Amazon Web Services](https://aws.amazon.com/tools/#sdk)

## HTTPS Query API<a name="asm-sdks_query-api"></a>

* provides
  * [programmatic access to](https://docs.aws.amazon.com/secretsmanager/latest/apireference/Welcome.html)
    * Secrets Manager
    * AWS
* allows
  * issuing HTTPS requests -- , via Secrets Manager endpoint, directly to the -- service 
* recommendations
  * 👀use better AWS SDK 👀
    * Reason: 🧠 SDK automatizes (!= manual) many useful tasks 🧠

## Secrets Manager endpoints<a name="endpoints"></a>

* allows
  * -- connecting programmatically to -- Secrets Manager 
* uses
  * by 
    * AWS SDKs
    * AWS CLI
* types
  * default endpoint
    * AWS Region-specific 
  * specified by you
* endpoints / support [Federal Information Processing Standard (FIPS) 140-2](http://aws.amazon.com/compliance/fips/) | some AWS Regions
* see [AWS documentation website](http://docs.aws.amazon.com/secretsmanager/latest/userguide/asm_access.html)