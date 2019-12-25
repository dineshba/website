---
title: "Which scheduler scheduled the scheduler in minikube ?"
date: 2019-12-25T13:10:48+05:30
---

In oneliner, minikube is a tool that makes it easy to run Kubernetes locally.

In `kube-system` namespace, we can result like below,

```bash
$ kubectl get pods -n kube-system -o name
pod/coredns-5c98db65d4-n9vvl
pod/coredns-5c98db65d4-p2hk7
pod/etcd-minikube
pod/kube-addon-manager-minikube
pod/kube-apiserver-minikube
pod/kube-controller-manager-minikube
pod/kube-proxy-zkw85
pod/kube-scheduler-minikube
pod/nginx-ingress-controller-657fd58d97-gp6rl
pod/storage-provisioner
```

Whenever we create pod(rs/deployments/sts/ds), `Scheduler` is the one which schedules this pods into any of the available node (in minikube, there is only one).
In minikube, `kube-scheduler-minikube` is the one who schedules. For long time, I wondered who scheduled the `kube-scheduler-minikube` pod to this minikube node. I assumed it as `magic` 🤪

Now, I understood how things are happening, here you go:

There is something called [static pods](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/) in kubernetes. Without  master components (`api-server`, `etcd`, `scheduler`, `controller-manager`), we can run pods in `kubelet` **directly**. Minikube used this to run master components and thus formed the cluster.

#### Proof:
```bash
$ minikube ssh
$ ps aux | grep kubelet
root      3035  3.9  4.6 1377692 91476 ?       Ssl  07:27   0:06 /var/lib/minikube/binaries/v1.15.2/kubelet --authorization-mode=Webhook --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --cgroup-driver=cgroupfs --client-ca-file=/var/lib/minikube/certs/ca.crt --cluster-dns=10.96.0.10 --cluster-domain=cluster.local --config=/var/lib/kubelet/config.yaml --container-runtime=docker --hostname-override=minikube --kubeconfig=/etc/kubernetes/kubelet.conf --node-ip=192.168.99.100 --pod-manifest-path=/etc/kubernetes/manifests
```

In the above example, kubelet is started with `--pod-manifest-path=/etc/kubernetes/manifests`

> Note: `--pod-manifest-path` is path to the directory containing static pod files to run.

```bash
$ ls -l /etc/kubernetes/manifests
-rw-r----- 1 root root 1532 Jan  1  0001 addon-manager.yaml.tmpl
-rw------- 1 root root 1990 Dec 25 07:27 etcd.yaml
-rw------- 1 root root 2893 Dec 25 07:27 kube-apiserver.yaml
-rw------- 1 root root 2262 Dec 25 07:27 kube-controller-manager.yaml
-rw------- 1 root root  990 Dec 25 07:27 kube-scheduler.yaml
```

Thus when kubelet started, it started the master components.