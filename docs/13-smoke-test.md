# Smoke Test

In this lab you will complete a series of tasks to ensure your Kubernetes cluster is functioning correctly.

## Data Encryption

In this section you will verify the ability to [encrypt secret data at rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#verifying-that-data-is-encrypted).

Create a generic secret:

```sh
kubectl create secret generic kubernetes-the-hard-way --from-literal="mykey=mydata"
```

Print a hexdump of the `kubernetes-the-hard-way` secret stored in etcd:

```sh
external_ip=$(aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=controller-0" \
  --output text --query 'Reservations[].Instances[].PublicIpAddress')

ssh -i kubernetes.id_rsa ubuntu@${external_ip}
```
Run below command in controller-0

```sh
sudo ETCDCTL_API=3 etcdctl get \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem\
  /registry/secrets/default/kubernetes-the-hard-way | hexdump -C
```

> output

```
00000000  2f 72 65 67 69 73 74 72  79 2f 73 65 63 72 65 74  |/registry/secret|
00000010  73 2f 64 65 66 61 75 6c  74 2f 6b 75 62 65 72 6e  |s/default/kubern|
00000020  65 74 65 73 2d 74 68 65  2d 68 61 72 64 2d 77 61  |etes-the-hard-wa|
00000030  79 0a 6b 38 73 3a 65 6e  63 3a 61 65 73 63 62 63  |y.k8s:enc:aescbc|
00000040  3a 76 31 3a 6b 65 79 31  3a b7 1a c3 4f c9 b7 c3  |:v1:key1:...O...|
00000050  65 3a a7 4a e0 68 7b b1  7f ab d2 a7 b1 c3 f5 fe  |e:.J.h{.........|
00000060  fc 60 3a b2 aa cd fc c7  d7 1c 8d 75 b0 bd 6c 36  |.`:........u..l6|
00000070  13 dc 0e 20 b9 c8 ed b6  2b ac 5c 17 10 ff 79 40  |... ....+.\...y@|
00000080  19 4f 14 3a 81 59 03 d8  dc 4f 4d a2 b3 1b 1c 4c  |.O.:.Y...OM....L|
00000090  dd 02 cf 32 1d b6 38 2c  6d 2e 40 4d 01 a0 ed 7b  |...2..8,m.@M...{|
000000a0  53 ff 8e 97 0f 0d 49 13  f1 bf c1 aa 72 26 fb d6  |S.....I.....r&..|
000000b0  99 7a c5 98 55 a8 c1 27  d1 5e ed 6a 39 24 16 d2  |.z..U..'.^.j9$..|
000000c0  60 12 63 cb 3e bf b3 77  5a 55 59 e0 61 03 16 2d  |`.c.>..wZUY.a..-|
000000d0  d6 13 22 7c 3e fe a1 56  77 a8 85 6e 28 4d 1d b4  |.."|>..Vw..n(M..|
000000e0  19 55 30 66 cd 08 84 94  cb a6 4d 2b 95 92 61 9b  |.U0f......M+..a.|
000000f0  3b ed 1f 2a af 55 c1 ad  db ed c8 80 03 28 c6 c4  |;..*.U.......(..|
00000100  78 ba 25 24 5d f3 a1 34  1f 32 0b 25 bd 13 ec 0b  |x.%$]..4.2.%....|
00000110  12 21 09 3e bd c8 ec 19  01 3c 21 a8 24 cc d3 19  |.!.>.....<!.$...|
00000120  cc 43 4e 95 b4 c9 2b 98  78 ba f2 1e 83 2d c3 98  |.CN...+.x....-..|
00000130  88 eb 85 af df 90 32 a2  a9 8a ec 54 5f 6f f6 01  |......2....T_o..|
00000140  7c ba 3a e9 b4 b3 3b e3  20 0a                    ||.:...;. .|
0000014a
```

The etcd key should be prefixed with `k8s:enc:aescbc:v1:key1`, which indicates the `aescbc` provider was used to encrypt the data with the `key1` encryption key.

## Deployments  - To be run on local laptop

In this section you will verify the ability to create and manage [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

Create a deployment for the [nginx](https://nginx.org/en/) web server:

```
kubectl create deployment nginx --image=nginx
```

List the pod created by the `nginx` deployment:

```
kubectl get pods -l app=nginx
```

> output

```
NAME                     READY     STATUS    RESTARTS   AGE
nginx-65899c769f-xkfcn   1/1       Running   0          15s
```

### Port Forwarding

In this section you will verify the ability to access applications remotely using [port forwarding](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/).

Retrieve the full name of the `nginx` pod:

```
POD_NAME=$(kubectl get pods -l app=nginx -o jsonpath="{.items[0].metadata.name}")
```

Forward port `8080` on your local machine to port `80` of the `nginx` pod:

```
kubectl port-forward $POD_NAME 8080:80
```

> output

```
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

In a new terminal make an HTTP request using the forwarding address:

```
curl --head http://127.0.0.1:8080
```

> output

```
HTTP/1.1 200 OK
Server: nginx/1.19.2
Date: Wed, 26 Aug 2020 17:22:59 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 11 Aug 2020 14:50:35 GMT
Connection: keep-alive
ETag: "5f32b03b-264"
Accept-Ranges: bytes
```

Switch back to the previous terminal and stop the port forwarding to the `nginx` pod:

```
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
^C
```

### Logs

In this section you will verify the ability to [retrieve container logs](https://kubernetes.io/docs/concepts/cluster-administration/logging/).

Print the `nginx` pod logs:

```
kubectl logs $POD_NAME
```

> output

```
127.0.0.1 - - [26/Aug/2020:17:22:59 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.29.0" "-"
```

### Exec

In this section you will verify the ability to [execute commands in a container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/#running-individual-commands-in-a-container).

Print the nginx version by executing the `nginx -v` command in the `nginx` container:

```
kubectl exec -ti $POD_NAME -- nginx -v
```

> output

```
nginx version: nginx/1.19.2
```

## Services

In this section you will verify the ability to expose applications using a [Service](https://kubernetes.io/docs/concepts/services-networking/service/).

Expose the `nginx` deployment using a [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport) service:

```
kubectl expose deployment nginx --port 80 --type NodePort
```

> The LoadBalancer service type can not be used because your cluster is not configured with [cloud provider integration](https://kubernetes.io/docs/getting-started-guides/scratch/#cloud-provider). Setting up cloud provider integration is out of scope for this tutorial.

Retrieve the node port assigned to the `nginx` service:

```
NODE_PORT=$(kubectl get svc nginx \
  --output=jsonpath='{range .spec.ports[0]}{.nodePort}')
```

Create a firewall rule that allows remote access to the `nginx` node port:

```
aws ec2 authorize-security-group-ingress \
  --group-id ${SECURITY_GROUP_ID} \
  --protocol tcp \
  --port ${NODE_PORT} \
  --cidr 0.0.0.0/0
```

Retrieve the external IP address of a worker instance:

```
INSTANCE_NAME=$(kubectl get pod $POD_NAME --output=jsonpath='{.spec.nodeName}')
```

If you deployed the cluster on US-EAST-1 use the command below:

```
EXTERNAL_IP=$(aws ec2 describe-instances \
    --filters "Name=network-interface.private-dns-name,Values=${INSTANCE_NAME}.ec2.internal" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
```

If you deployed the cluster on ANY OTHER region use this command:

```
EXTERNAL_IP=$(aws ec2 describe-instances \
    --filters "Name=network-interface.private-dns-name,Values=${INSTANCE_NAME}.${AWS_REGION}.compute.internal" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
```

Make an HTTP request using the external IP address and the `nginx` node port:

```
curl -I http://${EXTERNAL_IP}:${NODE_PORT}
```

> output

```
HTTP/1.1 200 OK
Server: nginx/1.19.2
Date: Wed, 26 Aug 2020 17:28:36 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 11 Aug 2020 14:50:35 GMT
Connection: keep-alive
ETag: "5f32b03b-264"
Accept-Ranges: bytes
```

## Untrusted Workloads

This section will verify the ability to run untrusted workloads using [gVisor](https://github.com/google/gvisor).

Create the `untrusted` pod:

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: untrusted
  annotations:
    io.kubernetes.cri.untrusted-workload: "true"
spec:
  containers:
    - name: webserver
      image: gcr.io/hightowerlabs/helloworld:2.0.0
EOF
```

### Verification

In this section you will verify the `untrusted` pod is running under gVisor (runsc) by inspecting the assigned worker node.

Verify the `untrusted` pod is running:

```
kubectl get pods -o wide
```

```
NAME                     READY     STATUS    RESTARTS   AGE       IP           NODE             NOMINATED NODE
busybox                  1/1       Running   0          5m        10.200.0.2   ip-10-0-1-20     <none>
nginx-64f497f8fd-l6b78   1/1       Running   0          3m        10.200.1.2   ip-10-0-1-21     <none>
untrusted                1/1       Running   0          8s        10.200.2.3   ip-10-0-1-22     <none>
```


Get the node name where the `untrusted` pod is running:

```
INSTANCE_NAME=$(kubectl get pod untrusted --output=jsonpath='{.spec.nodeName}')
```

If you deployed the cluster on US-EAST-1 use the command below:

```
INSTANCE_IP=$(aws ec2 describe-instances \
    --filters "Name=network-interface.private-dns-name,Values=${INSTANCE_NAME}.ec2.internal" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
```

If you deployed the cluster on ANY OTHER region use this command:

```
INSTANCE_IP=$(aws ec2 describe-instances \
    --filters "Name=network-interface.private-dns-name,Values=${INSTANCE_NAME}.${AWS_REGION}.compute.internal" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
```

SSH into the worker node:

```
ssh -i kubernetes.id_rsa ubuntu@${INSTANCE_IP}
```

List the containers running under gVisor:

```
sudo runsc --root  /run/containerd/runsc/k8s.io list
```

```
I0514 14:03:56.108368   14988 x:0] ***************************
I0514 14:03:56.108548   14988 x:0] Args: [runsc --root /run/containerd/runsc/k8s.io list]
I0514 14:03:56.108730   14988 x:0] Git Revision: 08879266fef3a67fac1a77f1ea133c3ac75759dd
I0514 14:03:56.108787   14988 x:0] PID: 14988
I0514 14:03:56.108838   14988 x:0] UID: 0, GID: 0
I0514 14:03:56.108877   14988 x:0] Configuration:
I0514 14:03:56.108912   14988 x:0]              RootDir: /run/containerd/runsc/k8s.io
I0514 14:03:56.109000   14988 x:0]              Platform: ptrace
I0514 14:03:56.109080   14988 x:0]              FileAccess: proxy, overlay: false
I0514 14:03:56.109159   14988 x:0]              Network: sandbox, logging: false
I0514 14:03:56.109238   14988 x:0]              Strace: false, max size: 1024, syscalls: []
I0514 14:03:56.109315   14988 x:0] ***************************
ID                                                                 PID         STATUS      BUNDLE                                                           CREATED                          OWNER
3528c6b270c76858e15e10ede61bd1100b77519e7c9972d51b370d6a3c60adbb   14766       running     /run/containerd/io.containerd.runtime.v1.linux/k8s.io/3528c6b270c76858e15e10ede61bd1100b77519e7c9972d51b370d6a3c60adbb   2018-05-14T14:02:34.302378996Z
7ff747c919c2dcf31e64d7673340885138317c91c7c51ec6302527df680ba981   14716       running     /run/containerd/io.containerd.runtime.v1.linux/k8s.io/7ff747c919c2dcf31e64d7673340885138317c91c7c51ec6302527df680ba981   2018-05-14T14:02:32.159552044Z
I0514 14:03:56.111287   14988 x:0] Exiting with status: 0
```

Get the ID of the `untrusted` pod:

```
POD_ID=$(sudo crictl -r unix:///var/run/containerd/containerd.sock pods --name untrusted -q)
```

Get the ID of the `webserver` container running in the `untrusted` pod:

```
CONTAINER_ID=$(sudo crictl -r unix:///var/run/containerd/containerd.sock ps -p ${POD_ID} -q)
```

Use the gVisor `runsc` command to display the processes running inside the `webserver` container:

```
sudo runsc --root /run/containerd/runsc/k8s.io ps ${CONTAINER_ID}
```

> output

```
I0514 14:05:16.499237   15096 x:0] ***************************
I0514 14:05:16.499542   15096 x:0] Args: [runsc --root /run/containerd/runsc/k8s.io ps 3528c6b270c76858e15e10ede61bd1100b77519e7c9972d51b370d6a3c60adbb]
I0514 14:05:16.499597   15096 x:0] Git Revision: 08879266fef3a67fac1a77f1ea133c3ac75759dd
I0514 14:05:16.499644   15096 x:0] PID: 15096
I0514 14:05:16.499695   15096 x:0] UID: 0, GID: 0
I0514 14:05:16.499734   15096 x:0] Configuration:
I0514 14:05:16.499769   15096 x:0]              RootDir: /run/containerd/runsc/k8s.io
I0514 14:05:16.499880   15096 x:0]              Platform: ptrace
I0514 14:05:16.499962   15096 x:0]              FileAccess: proxy, overlay: false
I0514 14:05:16.500042   15096 x:0]              Network: sandbox, logging: false
I0514 14:05:16.500120   15096 x:0]              Strace: false, max size: 1024, syscalls: []
I0514 14:05:16.500197   15096 x:0] ***************************
UID       PID       PPID      C         STIME     TIME      CMD
0         1         0         0         14:02     40ms      app
I0514 14:05:16.501354   15096 x:0] Exiting with status: 0
```

# Check images/pods/containers on worker nodes using crictl

Log in to a worker node. You can do this on all 3 workers to see the resources on each of them:

```sh
external_ip=$(aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=worker-0" \
  --output text --query 'Reservations[].Instances[].PublicIpAddress')

ssh -i kubernetes.id_rsa ubuntu@${external_ip}
```
Run following commands and check output

```sh
sudo crictl -r unix:///var/run/containerd/containerd.sock images
```

Output
```sh
IMAGE                                                  TAG                 IMAGE ID            SIZE
gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64   1.14.7              5feec37454f45       10.9MB
gcr.io/google_containers/k8s-dns-kube-dns-amd64        1.14.7              5d049a8c4eec9       13.1MB
gcr.io/google_containers/k8s-dns-sidecar-amd64         1.14.7              db76ee297b859       11.2MB
k8s.gcr.io/pause                                       3.1                 da86e6ba6ca19       317kB
```

```sh
sudo crictl -r unix:///var/run/containerd/containerd.sock pods
```

Output
```sh
POD ID              CREATED             STATE               NAME                        NAMESPACE           ATTEMPT
9a304a19557f7       2 hours ago         Ready               kube-dns-864b8bdc77-c5vc2   kube-system         0
```

```sh
sudo crictl -r unix:///var/run/containerd/containerd.sock ps
```

Output
```sh
CONTAINER ID        IMAGE                                                                     CREATED             STATE               NAME                ATTEMPT
611bfea53997d       sha256:db76ee297b8597fc007b23a90619314b8405bb1df6dcad189df0a123a09e7ecc   2 hours ago         Running             sidecar             0
824f26368efc0       sha256:5feec37454f45d060c5f528c7d0bd4958df39e7ffd2e65ae42aae68bf78f69a5   2 hours ago         Running             dnsmasq             0
f3d35b783af1e       sha256:5d049a8c4eec92b21ca4be399c260166d96569a1a52d497f4a0365bb55c1a18c   2 hours ago         Running             kubedns             0
```


Next: [Cleaning Up](14-cleanup.md)
