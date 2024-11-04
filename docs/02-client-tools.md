# Installing the Client Tools

In this lab you will install the command line utilities required to complete this tutorial: [cfssl](https://github.com/cloudflare/cfssl), [cfssljson](https://github.com/cloudflare/cfssl), and [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl).


## Install CFSSL

The `cfssl` and `cfssljson` command line utilities will be used to provision a [PKI Infrastructure](https://en.wikipedia.org/wiki/Public_key_infrastructure) and generate TLS certificates.

Download and install `cfssl` and `cfssljson`:

### OS X

```
curl -o cfssl https://github.com/cloudflare/cfssl/releases/download/v1.6.4/cfssl_1.6.4_linux_amd64
curl -o cfssljson https://github.com/cloudflare/cfssl/releases/download/v1.6.4/cfssljson_1.6.4_linux_amd64
```

```
chmod +x cfssl cfssljson
```

```
sudo mv cfssl cfssljson /usr/local/bin/
```

Some OS X users may experience problems using the pre-built binaries in which case [Homebrew](https://brew.sh) might be a better option:

```
brew install cfssl
```

### Linux

Find the architecture using the following command (example output given).
```
$ lscpu | grep -i 'model name\|arch'
Architecture:                       x86_64
Model name:                         AMD EPYC 7763 64-Core Processor
```

We could set the [version](https://github.com/cloudflare/cfssl/releases/tag/v1.6.5) and architecture as variables, download the tools and rename those.
```
VERSION="1.6.5"
ARCH="amd64"

wget -q --show-progress \ 
  https://github.com/cloudflare/cfssl/releases/download/v${VERSION}/cfssl_${VERSION}_linux_${ARCH} \ 
  https://github.com/cloudflare/cfssl/releases/download/v${VERSION}/cfssljson_${VERSION}_linux_${ARCH}

mv cfssl_${VERSION}_linux_${ARCH} cfssl
mv cfssljson_${VERSION}_linux_${ARCH} cfssljson
```

Make those binaries executable.
```
chmod +x cfssl cfssljson
```

Move them to the `/usr/local/bin` so that they could be run from anywhere on the system. 
```
sudo mv cfssl cfssljson /usr/local/bin/
```

### Verification

Verify `cfssl` and `cfssljson` version 1.4.1 or higher is installed:

```
cfssl version
```

> output

```
Version: 1.4.1
Runtime: go1.12.12
```

```
cfssljson --version
```
```
Version: 1.4.1
Runtime: go1.12.12
```

## Install kubectl

The `kubectl` command line utility is used to interact with the Kubernetes API Server. Download and install `kubectl` from the official release binaries:

### OS X

```
curl -o kubectl https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/darwin/amd64/kubectl
```

```
chmod +x kubectl
```

```
sudo mv kubectl /usr/local/bin/
```

### Linux

```
wget https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl
```

```
chmod +x kubectl
```

```
sudo mv kubectl /usr/local/bin/
```

### Verification

Verify `kubectl` version 1.21.0 or higher is installed:

```
kubectl version --client
```

> output

```
Client Version: version.Info{Major:"1", Minor:"21", GitVersion:"v1.21.0", GitCommit:"cb303e613a121a29364f75cc67d3d580833a7479", GitTreeState:"clean", BuildDate:"2021-04-08T16:31:21Z", GoVersion:"go1.16.1", Compiler:"gc", Platform:"linux/amd64"}
```

Next: [Provisioning Compute Resources](03-compute-resources.md)
