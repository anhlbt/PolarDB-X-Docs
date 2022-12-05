# Deploy the cluster via Kubernetes

This article describes how to create a simple Kubernetes cluster, deploy the PolarDB-X Operator, and use the Operator to deploy a full PolarDB-X cluster.

> Note: The deployment instructions in this article are for testing purposes only and should not be used directly in a production environment.

# Create a Kubernetes test cluster

This section mainly introduces how to use [minikube](https://minikube.sigs.k8s.io/docs/start/) to create a Kubernetes test cluster. You can also use Alibaba Cloud’s [Container Service ACK](https://www .aliyun.com/product/kubernetes) to create a Kubernetes cluster, and follow the tutorial to deploy PolarDB-X Operator and PolarDB-X cluster.

## Create a Kubernetes cluster using minikube

[minikube](https://minikube.sigs.k8s.io/docs/start/) is a tool maintained by the community for quickly creating Kubernetes test clusters, suitable for testing and learning Kubernetes. A Kubernetes cluster created using minikube can run in a container or a virtual machine. In this section, the creation of Kubernetes on CentOS 8.2 is taken as an example.

> Note: If you deploy minikube on other operating systems such as macOS or Windows, some steps may be slightly different.

Before deployment, please ensure that minikube and Docker have been installed and meet the following requirements:

+ The machine specification is not less than 4c8g
+ minikube >= 1.18.0
+ docker >= 1.19.3

minikube requires a non-root account for deployment. If you access the machine with a root account, you need to create a new account.

```bash
$ useradd -ms /bin/bash galaxykube
$ usermod -aG docker galaxykube
```

If you use another account, please add it to the docker group as above to ensure that it can directly access docker.

Use su to switch to the account `galaxykube`,

```bash
$ are galaxycubes
```

Execute the following command to start a minikube,

```bash
minikube start --cpus 4 --memory 7960 --image-mirror-country cn --registry-mirror=https://docker.mirrors.ustc.edu.cn
```

> Note: Here we use Alibaba Cloud’s minikube image source and the docker image source provided by USTC to speed up the pull of the image.

If everything went fine, you should see output similar to the one below.

```bash
😄  minikube v1.23.2 on Centos 8.2.2004 (amd64)
✨  Using the docker driver based on existing profile
❗  Your cgroup does not allow setting memory.
▪ More information: https://docs.docker.com/engine/install/linux-postinstall/#your-kernel-does-not-support-cgroup-swap-limit-capabilities
❗  Your cgroup does not allow setting memory.
▪ More information: https://docs.docker.com/engine/install/linux-postinstall/#your-kernel-does-not-support-cgroup-swap-limit-capabilities
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🤷  docker "minikube" container is missing, will recreate.
🔥  Creating docker container (CPUs=4, Memory=7960MB) ...
> kubeadm.sha256: 64 B / 64 B [--------------------------] 100.00% ? p/s 0s
> kubelet.sha256: 64 B / 64 B [--------------------------] 100.00% ? p/s 0s
> kubectl.sha256: 64 B / 64 B [--------------------------] 100.00% ? p/s 0s
> kubeadm: 43.71 MiB / 43.71 MiB [---------------] 100.00% 1.01 MiB p/s 44s
> kubectl: 44.73 MiB / 44.73 MiB [-------------] 100.00% 910.41 KiB p/s 51s
> cube: 146.25 MiB / 146.25 MiB [ -------------] 100.00% 2.71 MiB p/s 54p

▪ Generating certificates and keys ...
▪ Booting up control plane ...
▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
▪ Using image registry.cn-hangzhou.aliyuncs.com/google_containers/storage-provisioner:v5 (global image repository)
🌟  Enabled addons: storage-provisioner, default-storageclass
💡  kubectl not found. If you need it, try: 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

At this point minikube is running normally. minikube will automatically set up the kubectl configuration file, if you have installed kubectl before, you can use kubectl to access the cluster now:

```bash
$ kubectl cluster-info
kubectl cluster-info
Kubernetes control plane is running at https://192.168.49.2:8443
CoreDNS is running at https://192.168.49.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

If kubectl is not installed, minikube also provides subcommands to use kubectl:

```bash
$ minikube kubectl -- cluster-info
Kubernetes control plane is running at https://192.168.49.2:8443
CoreDNS is running at https://192.168.49.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

> Note: The minikube kubectl subcommand needs to add "--" before the kubectl parameter. If you use the bash shell, you can use alias kubectl="minikube kubectl -- " to set the shortcut command. The following will use the kubectl command to operate.

Now we can start deploying the PolarDB-X Operator!

> After testing, execute minikube delete to destroy the cluster.

# Deploy PolarDB-X Operator

Before getting started, make sure the following prerequisites are met:

+ Have prepared a running Kubernetes cluster and ensured
+ Cluster version >= 1.18.0
+ At least 2 CPUs to allocate
+ At least 4GB of allocatable memory
+ At least 30GB of disk space
+ kubectl has been installed to access the Kubernetes cluster
+ Installed [Helm 3](https://helm.sh/docs/intro/install/)


First create a namespace called `polardbx-operator-system`,

```bash
$ kubectl create namespace polardbx-operator-system
```

Execute the following command to install PolarDB-X Operator.

```bash
$ helm install --namespace polardbx-operator-system polardbx-operator https://github.com/ApsaraDB/galaxykube/releases/download/v1.2.1/polardbx-operator-1.2.1.tgz
```

You can also install from PolarDB-X's Helm Chart repository:

```shell
helm repo add polardbx https://polardbx-charts.oss-cn-beijing.aliyuncs.com
helm install --namespace polardbx-operator-system polardbx-operator polardbx/polardbx-operator
```

Expect to see output like this:

```bash
NAME: polardbx-operator
LAST DEPLOYED: Sun Oct 17 15:17:29 2021
NAMESPACE: polardbx-operator-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
polardbx-operator is installed. Please check the status of components:

kubectl get pods --namespace polardbx-operator-system

Now have fun with your first PolarDB-X cluster.

Here's the manifest for quick start:

```yaml
apiVersion: polardbx.aliyun.com/v1
kind: PolarDBXCluster
Metata:
name: quick-start
annotations:
polardbx/topology-mode-guide: quick-start
```

Check the running status of the PolarDB-X Operator components and wait for them to enter the Running state:

```bash
$ kubectl get pods --namespace polardbx-operator-system
NAME                                           READY   STATUS    RESTARTS   AGE
polardbx-controller-manager-6c858fc5b9-zrhx9   1/1     Running   0          66s
polardbx-hpfs-d44zd                            1/1     Running   0          66s
polardbx-tools-updater-459lc                   1/1     Running   0          66s
```

Congratulations! The PolarDB-X Operator has been installed, and now you can start deploying the PolarDB-X cluster!

# Deploy PolarDB-X cluster

Now let's quickly deploy a PolarDB-X cluster, which contains 1 GMS node, 1 CN node, 1 DN node and 1 CDC node. Execute the following command to create such a cluster:

```bash
echo "apiVersion: polardbx.aliyun.com/v1
kind: PolarDBXCluster
Metata:
name: quick-start
annotations:
polardbx/topology-mode-guide: quick-start" | kubectl apply -f -
```

You will see the following output:

```bash
polardbxcluster.polardbx.aliyun.com/quick-start created
```

Use the following command to check the creation status:

```bash
$ kubectl get polardbxcluster -w
NAME          GMS   CN    DN    CDC   PHASE      DISK   AGE
quick-start   0/1   0/1   0/1   0/1   Creating          35s
quick-start   1/1   0/1   1/1   0/1   Creating          93s
quick-start   1/1   0/1   1/1   1/1   Creating          4m43s
quick-start   1/1   1/1   1/1   1/1   Running    2.4 GiB   4m44s
```

When PHASE shows Running, the PolarDB-X cluster has been deployed! Congratulations, you are now ready to connect and experience the PolarDB-X distributed database!

# Connect PolarDB-X cluster

PolarDB-X supports the MySQL transport protocol and most syntaxes, so you can use the mysql command line tool to connect to PolarDB-X for database operations.

Before starting, make sure you have the mysql command line tools installed.

## Forward the access port of PolarDB-X

When creating a PolarDB-X cluster, the PolarDB-X Operator will also create an access service for the cluster, which is of the ClusterIP type by default. Use the following command to view the services used for access:

```bash
$ kubectl get svc quick-start
```

Expected output:

```bash
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
quick-start   ClusterIP   10.110.214.223   <none>        3306/TCP,8081/TCP   5m25s
```

We use the port-forward name provided by kubectl to forward port 3306 of the service to the local, and keep the forwarding process alive.

```bash
$ kubectl port-forward svc/quick-start 3306
```

## Connect PolarDB-X cluster

The Operator will create an account polardbx_root for the PolarDB-X cluster by default, and store the password in secret.

Use the following command to view the password of the polardbx_root account:

```bash
$ kubectl get secret quick-start -o jsonpath="{.data['polardbx_root']}" | base64 -d - | xargs echo "Password: "
Password:  bvp9wjxx
```

Keep port-forward running, reopen a terminal, and execute the following command to connect to the cluster:

```bash
$ mysql -h127.0.0.1 -P3306 -upolardbx_root -pbvp9wjxx
```

Expected output:

```bash
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 6
Server version: 5.6.29 Tddl Server (ALIBABA)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

Congratulations! You have successfully deployed and connected to a PolarDB-X distributed database cluster, now you can start to experience the power of distributed databases!

# Destroy the PolarDB-X cluster

After testing, you can destroy the PolarDB-X cluster with the following command.

```bash
$ kubectl delete polardbxcluster quick-start
```

Check again to make sure the deletion is complete

```bash
$ kubectl get polardbxcluster quick-start
```

# Uninstall PolarDB-X Operator

Use the following command to uninstall PolarDB-X Operator.

```bash
$ helm uninstall --namespace polardbx-operator-system polardbx-operator
```

Uninstalling Helm will not delete the corresponding custom resource CRD. Use the following command to view and delete the custom resource corresponding to PolarDB-X:

```bash
$ kubectl get crds | grabbed polardbx.aliyun.com
Pollardbacksclusters.pollardbacks.eyeon.com 2021-10-17T07:17:I notice
xstores.polardbx.aliyun.com 2021-10-17T07:17:27Z

$ kubectl delete crds polardbxclusters.polardbx.aliyun.com xstores.polardbx.aliyun.com
```