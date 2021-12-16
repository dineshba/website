---
title: "Identity Federation"
date: 2021-12-15T00:50:33+05:30
---

Before Identity Federation, lets talk about How are we accessing resources in Cloud

### Accessing resources in Cloud

![wif_1](/wif_1.png)

> Note: Application needs **access** to consume the above resources

##### Examples of above diagram
- Application running in on-prem accessing GCP resources (like gcs bucket, container registry, S3, vault, etc)
- Application running in other cloud accessing GCP resources
- Github actions trying to deploy/create resources in Azure
- Jenkins running in Kubernetes trying to deploy things to AWS
- Application running in GCP cloud function trying to access to aws resource

#### Using Identities of that cloud

![wif_2](/wif_2.png)

- Create an Identity
- Attach permissions
- Create key/secret/token for that identity
- Share the key with application

#### What are the problems of using cloud identity outside that cloud ?

- If there is expiration for key/secret/token, then **rotating** it
    - create new keys in cloud
    - deploy/restart the all the applications (with their replicas) in proper order
    - remove the old keys in cloud
- If no expiration, **static keys**
- Not only the configured application, **anyone with key can access it**
    - Developer can copy the key and use it from their local machine
    - Developer can share the keys with co-workers (co-worker can also become ex-co-worker)

> It became Secret management problem from Identity/Access management problem

#### How will we do it, if the application is in the same cloud ?

![wif_3](/wif_3.png)

- Attach Identity to that resource
    - Eg:
        - [Attaching ServiceAccount to GCP instance](https://cloud.google.com/compute/docs/access/create-enable-service-accounts-for-instances)
        - [Attaching IAM instance profile to the EC2 instance](https://aws.amazon.com/premiumsupport/knowledge-center/ec2-instance-access-s3-bucket/)

> Keyless authentication is the key 🙃

### Identity Federation

- What if we are able to use same identity across clouds.
- What if we are able to use one identity to exchange the another identity for limited period

That is what **Identity federation** is.

![wif_4](/wif_4.png)

1. Application get Identity key/secret/token from its own local Identity Provider
    - Each cloud will have its own Identity Provider
    - Example custom Identity Provider: https://github.com/dexidp/dex
2. Application sends this Identity key/secret/token to the destination cloud's Token Provider
3. Token Provider will validate this Identity key/secret/token against application's Identity Provider
    - Application's Identity Provider should be preconfigured in cloud's Token Provider
    - Example Token Providers
        - [AWS Security Token Service](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html)
        - [GCP Security Token Service](https://cloud.google.com/iam/docs/reference/sts/rest)
4. Token Provider will issue a short lived token to Application
    - The token will have permission/role based on preconfigured values
5. Now, application can use to access resources in cloud

#### Examples:
- Deploy infra in google cloud using github actions (with github token). [Refer](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-google-cloud-platform#adding-a-google-cloud-workload-identity-provider)
- Deploy infra in aws cloud using github actions (with github token). [Refer](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services)
- Kubernetes pods access Vault using kubernetes service accounts. [Refer](https://www.vaultproject.io/docs/auth/kubernetes)
- Assume an AWS Role from a Google Cloud Instances without IAM keys. [Refer](https://blog.doit-intl.com/assume-an-aws-role-from-a-google-cloud-without-using-iam-keys-55012b0fa74a)
- Allow EC2 instance to access google container registry. [Refer](https://cloud.google.com/iam/docs/configuring-workload-identity-federation#aws_1)
- Allow particular instance of EC2 instance to talk to vault. [Refer](https://www.vaultproject.io/docs/auth/aws)