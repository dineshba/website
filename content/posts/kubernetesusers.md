---
title: "There are no users in Kubernetes"
date: 2020-06-20T22:58:45+05:30
---

#### Where is user information stored in kubernetes ?
Kubernetes will not store the user details.

#### Then how kubernetes is able to do the Authentication ?
- Using CA certificates
    - We can configure to use CA authority
    - Create CA certificates with user name and sign it with kubernetes

- Using external systems
    - As it is deligating this authentication to external systems, we can easily onboard users into the kubernetes. Eg: Lets say your organization have Okta based SSO, then you can easily integrate that to kubernetes

![auth.png](/auth.png)

#### How kubernetes components are talking securely ?
- Using CA certificates
    - If you running in minikube or using kubeadm, you see the volume mount of cert files to the pods in kube-system namespace.
        - Eg: scheduler pod will have scheduler.conf file mounted

#### Can we share this config file to everybody?
- We can definitely share. Anybody with CA certificate can talk to kubernetes and get authenticated
- But, it is very difficult to invalidate the CA certificate. So if cert is leaked, in order to invalidate, we have to reset CA authority in master and change all the CA certificates.

#### Can we use CA method to authenticate machines to talk to kubernetes ?
- Yes we can do it
- But, it is very difficult to invalidate the CA certificate. So if cert is leaked, in order to invalidate, we have to reset CA authority in master and change all the CA certificates.

####  Then how we can give permission for machine/process/bot to talk to kubernetes ?
- We have another type of special user called `ServiceAccount`
    - This serviceAccount will be stored in kubernetes (etcd)
    - We can easily create serviceAccount `kubectl create sa <sa-name>`
    - It will automatically create a secret (jwt token), using this token we can talk to kubernetes
![sa.png](/sa.png)

#### How authorization is happening in kubernetes ?
- Using RBAC
    - This information will be stored inside the kubernetes(etcd) as kubernetes objects
    - There are 4 objects available
        - Role (Resources + permission)
            - eg: resource = pod, permission = list,get
        - RoleBinding (Role + User)
            - eg: readonlyRole + dinesh or readonlyRole + sa
            - rolebinding can be for a user or for a service account
        - ClusterRole (Same as Role but for all namespaces)
        - ClusterRoleBinding (Same as Role but for all namespaces)

##### Eg:
Create a role to get,list,watch pods
```
cat <<EOF | kubectl apply -f -
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: podReader
  namespace: default
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
  - list
EOF
```
Create a role binding for the user `dinesh` and for service account `dinesh` with role `podReader`.
```
cat <<EOF | kubectl apply -f -
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dinesh-podReader
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: podReader
subjects:
- kind: ServiceAccount
  name: dinesh
  namespace: default
- kind: User
  name: dinesh
  namespace: default
EOF
```

Once the user is authenticated, based on the role, they/it can access the resource.