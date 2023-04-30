---
title: "Encrypt Kubernetes data at rest"
date: 2023-04-22T13:32:13+05:30
---

Kubernetes stores all of its data in etcd. Apiserver is the only component which reads and writes data into etcd.

![k8s-cluster.png](/k8s-cluster.png)

So, before storing data into etcd, we can ask apiserver to encrypt the data and while it retrieve data from etcd, we can ask apiserver to decrypt the data.

When we start api-server, we have to provide the encryption config with below flag:
```yaml
--encryption-provider-config=/etc/kubernetes/enc/encryptionconfig.yaml
# --encryption-provider-config-automatic-reload=true # optional
```

> Note: If apiserver is running as pod, remember to mount file `/etc/kubernetes/enc/encryptionconfig.yaml` to apiserver pod

### Sample encryptionconfig.yaml
```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aesgcm:
          keys:
            - name: key1
              secret: c2VjcmV0IGlzIHNlY3VyZQ==
      - identity: {}
```              

- Complete example file: https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#understanding-the-encryption-at-rest-configuration
- Available providers: https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#providers

> We are using standard method aesgcm and static keys in above example

> `identity: {}` is very important if you already have a secret before configuring encryption. Details in next section

## Key points to note
- Only first provider is used for encryption
- Each provider will be used to attempt for decryption until success
- If no provider can read the stored data due to a mismatch in format or secret key, we can see error in api-server `failed to list *core.Secret: unable to transform key "/registry/secrets/<namespace>/<secret-name>"`
- If we lost the encryption key, only option is to delete the key directly in etcd

> Note: Without having each provider to attempt decryption, we cannot achieve key rotation and we cannot change from one encryption method to another.
## External Key Management Service

Encrypting secret data with a locally managed key protects against an etcd compromise, but it fails to protect against a host compromise.

Instead of using local keys, we can use external key management service (KMS).

Almost each cloud provider is their own KMS solution.
#### Example Key Management Service:
- https://learn.microsoft.com/en-us/azure/key-vault/general/
- https://cloud.google.com/security-key-management
- https://docs.aws.amazon.com/kms/latest/developerguide/overview.html

If we want to use it api-server, then api-server should know
- How to authenticate against this cloud ?
- How to interact with this encrypt/decrypt API ?

So, if we put this logic in api-server, it will not scale. So kubernetes/kubernetes-community provides a **common interface and anybody who adhers to it can act as KMS provider for api-server**. https://kubernetes.io/docs/tasks/administer-cluster/kms-provider/#enabling-the-kms-supported-by-your-cloud-provider

#### Example KMS providers
- https://github.com/Azure/kubernetes-kms
- https://github.com/GoogleCloudPlatform/k8s-cloudkms-plugin
- https://github.com/kubernetes-sigs/aws-encryption-provider

#### Sample config using KMS provider

```yaml
apiVersion: v1
kind: EncryptionConfig
resources:
- providers:
  - kms:
      cachesize: 100
      endpoint: unix:///var/run/kmsPlugin.sock
      name: kms-plugin
  resources:
  - secrets
```

#### Develop your own KMS provider
We can use any service as KMS provider if it expose grpc methods with signature defined in https://kubernetes.io/docs/tasks/administer-cluster/kms-provider/#developing-a-kms-plugin-grpc-server

Your kms provider should create the socket file and expose methods to encrypt/decrypt

## Commands used to test encryption

```sh
# list keys in etcd
kubectl exec -it -n kube-system <etcd-pod> -- sh -c "ETCDCTL_API=3 etcdctl --endpoints 127.0.0.1:2379 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key get / --prefix --keys-only"

# show one key's value
kubectl exec -it -n kube-system <etcd-pod> -- sh -c "ETCDCTL_API=3 etcdctl --endpoints 127.0.0.1:2379 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key get /registry/secrets/<namespace>/<secret-name>"
# it should show as encrypted data. If it shows data in readable form, encryption at rest is not configured properly
```

## References
- https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/
- https://kubernetes.io/docs/tasks/administer-cluster/kms-provider/
- https://kubernetes.io/docs/tasks/administer-cluster/kms-provider/#switching-from-a-local-encryption-provider-to-the-kms-provider
- https://kubernetes.io/docs/tasks/administer-cluster/kms-provider/#disabling-encryption-at-rest
- https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#rotating-a-decryption-key