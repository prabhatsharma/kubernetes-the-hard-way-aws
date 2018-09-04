# Deploying the DNS Cluster Add-on

In this lab you will deploy the [DNS add-on](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) which provides DNS based service discovery to applications running inside the Kubernetes cluster.

## The DNS Cluster Add-on

Deploy the `kube-dns` cluster add-on:

```
kubectl create -f https://storage.googleapis.com/kubernetes-the-hard-way/kube-dns.yaml
```

> output

```
service "kube-dns" created
serviceaccount "kube-dns" created
configmap "kube-dns" created
deployment.extensions "kube-dns" created
```

List the pods created by the `kube-dns` deployment:

```
kubectl get pods -l k8s-app=kube-dns -n kube-system
```

> output

```
NAME                        READY     STATUS    RESTARTS   AGE
kube-dns-3097350089-gq015   3/3       Running   0          20s
```

## Verification

Create a `dnsutils` pod
(Not creating a busybox as it has [issue#356](https://github.com/kelseyhightower/kubernetes-the-hard-way/issues/356)  with DNS resolution):

```
kubectl run netutils --image=hiprabhat/netutils:2 --restart=Never -- sleep 3600
```

Verify that the pod is running:

```sh
kubectl get pod netutils
```

Output:
```
NAME       READY     STATUS    RESTARTS   AGE
netutils   1/1       Running   0          45s
```

Execute a DNS lookup for the `kubernetes` service inside the `dnsutils` pod:

```
kubectl exec -it netutils -- nslookup kubernetes
```

> output

```
Server:		10.32.0.10
Address:	10.32.0.10#53

Name:	kubernetes.default.svc.cluster.local
Address: 10.32.0.1
```

Next: [Smoke Test](13-smoke-test.md)
