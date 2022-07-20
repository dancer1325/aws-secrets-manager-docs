# How AWS Direct Connect uses AWS Secrets Manager<a name="integrating_how-services-use-secrets_directconnect"></a>

AWS Direct Connect links your internal network to an AWS Direct Connect location over a standard Ethernet fiber\-optic cable\. With this connection, you can create virtual interfaces directly to public AWS services\. 

AWS Direct Connect stores a connectivity association key name and connectivity association key pair \(CKN/CAK pair\) in a secret with the prefix `directconnect`\. The cost of the secret is included with the charge for AWS Direct Connect\. To update the secret, you must use AWS Direct Connect rather than Secrets Manager\. For more information, see [Associate a MACsec CKN/CAK with a LAG ](https://docs.aws.amazon.com/directconnect/latest/UserGuide/associate-key-lag.html) in the *AWS Direct Connect User Guide*\.