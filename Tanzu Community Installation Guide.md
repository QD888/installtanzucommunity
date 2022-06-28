# VMware Tanzu Community Installation Cookbook


By Leo Cao, Jun 23, 2022

-----------

## Environment Preparation

We are going to install Tanzu Community with Docker on Ubuntu 21.04.


**Environment Requirements**

**Important**: To ensure github.com, k8s.io and docker.io can be accessed from your labrary network.

 Item                   | Version
 -----------------------|-------------
 OS                     | Ubuntu 21.04
 Tanzu Community        | tce-linux-amd64-v0.12.0.tar
 Docker Engine          | 20.10.14

## Installation Process
```mermaid
graph LR
A(Start) -->B(Install Ubuntu VM)
B(Install Ubuntu VM) --> C(Install kubectl) 
C(Install kubectl) --> D(Install Docker)
D(Install Docker) --> E(Intall Tanzu Community)
E(Intall Tanzu Community) --> F(End)
```
## Installation Steps

### 1 Install Ubuntu VM

#### 1.1 Insatll Hypervior 

Install VMware Workstation on Windows Desktop or VMware Fusion on Mac Book.

#### 1.2 Deploy Ubuntu VM

Deploy one VM with below configuraitons:

Image of Ubuntu 21.04: https://old-releases.ubuntu.com/releases/21.04/ubuntu-21.04-live-server-amd64.iso

 Resources   | Size/Version
 ------------|-------------
 OS          | Ubuntu 64 bit
 vCPU        | 4
 Memeory     | 8 GB 
 Disk        | 50 GB

#### 1.3 Post-installation preparation

Install network tools. 
```
sudo apt update
sudo apt install net-tools 
sudo apt install curl
```

### 2. Install ```kubectl```

Install ```kubectl``` binary with curl on Ubuntu.
 For more information, see Install and Set Up kubectl on Linux in the Kubernetes documentation.https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

#### 2.1 Download the latest release with the command: 

```
curl -LO https://dl.k8s.io/release/v1.20.1/bin/linux/amd64/kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

#### 2.2 Validate the binary (optional)

Download the kubectl checksum
```
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
```

Validate the kubectl binary against the checksum file:

```
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
```

If valid, the output is:
```
kubectl: OK
```

#### 2.3 Install Kubectl 

```
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

### 3.Install Docker
You must download and install the latest version of docker. For more information, see Install Docker Engine in the Docker documentation.https://docs.docker.com/engine/install/

Install Docker Engine on Ubuntu using the repository.

#### 3.1 Set up the repository
1. Update the ```apt``` package index and install packages to allow ```apt``` to use a repository over HTTPS:
```
sudo apt-get update
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```
2. Add Docker’s official GPG key:
```
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

3. Use the following command to set up the repository:
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
#### 3.2 Install Docker Engine
1. Update the apt package index, and install the latest version of Docker Engine, containerd, and Docker Compose, or go to the next step to install a specific version:

```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

2. Verify that Docker Engine is installed correctly by running the ```hello-world``` image.
```
sudo docker run hello-world
```
This command downloads a test image and runs it in a container. When the container runs, it prints a message and exits.

### 4.Install Tanzu Community

#### 4.1 Install Tanzu CLI 
Curl GitHub release.

1. curl Download the release for Linux via web browser.
```
wget https://github.com/vmware-tanzu/community-edition/releases/download/v0.12.0/tce-linux-amd64-v0.12.0.tar.gz
```

2. Unpack the release.
```
tar xzvf ~/<DOWNLOAD-DIR>/tce-linux-amd64-v0.12.0.tar.gz
```

3. Run the install script (make sure to use the appropriate directory for your platform).
```
cd tce-linux-amd64-v0.12.0
./install.sh
```

#### 4.2 Deploy a Tanzu Management Cluster to Docker
#### 4.2.1 Prerequisites

1. The following additional configuration is needed for the Docker engine on your local client machine (with no other containers running):
**6 GB of RAM**
**15 GB of local machine disk storage for images**
**4 CPUs**

2. Manage Docker as a non-root user
**Important**
To create the docker group and add your user:

a. Create the docker group
```
$ sudo groupadd docker
```

b. Add your user to the docker group.
```
$ sudo usermod -aG docker $USER
```

c. Log out and log back in so that your group membership is re-evaluated.
```
$ newgrp docker
```

d. Verify that you can run docker commands without sudo.
```
$ docker run hello-world
```

3. Configure Docker to start on boot
Most current Linux distributions (RHEL, CentOS, Fedora, Debian, Ubuntu 16.04 and higher) use systemd to manage which services start when the system boots. On Debian and Ubuntu, the Docker service is configured to start on boot by default. To automatically start Docker and Containerd on boot for other distros, use the commands below:
```
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```


4. When using the Docker (CAPD) provider, the load balancer image (HA Proxy) is pulled from DockerHub. DockerHub limits pulls per user and this can especially impact users who share a common IP, in the case of NAT or VPN. If DockerHub rate-limiting is an issue in your environment, you can pre-pull the load balancer image to your machine by running the following command.

```
$ docker pull kindest/haproxy:v20210715-a6da3463
```

#### 4.2.2 Deploy a Management Cluster

1. Initialize the Tanzu Community Edition installer interface.
```
tanzu management-cluster create --ui
```
2. Complete the configuration steps in the installer interface for Docker and create the management cluster. The following configuration settings are recommended:
The Kubernetes Network Settings are auto-filled with a default CNI Provider and Cluster Service CIDR.
Docker Proxy settings are experimental and are to be used at your own risk.
We will have more complete tanzu cluster bootstrapping documentation available here in the near future.
If you ran the prune command in the previous step, expect this to take some time, as it’ll download an image that is over 1GB.

3. Validate the management cluster started:
```
tanzu management-cluster get
```
The output should look similar to the following:
```
NAME                                               READY  SEVERITY  REASON  SINCE  MESSAGE
/tkg-mgmt-docker-20210601125056                                                                 True                     28s
├─ClusterInfrastructure - DockerCluster/tkg-mgmt-docker-20210601125056                          True                     32s
├─ControlPlane - KubeadmControlPlane/tkg-mgmt-docker-20210601125056-control-plane               True                     28s
│ └─Machine/tkg-mgmt-docker-20210601125056-control-plane-5pkcp                                  True                     24s
│   └─MachineInfrastructure - DockerMachine/tkg-mgmt-docker-20210601125056-control-plane-9wlf2
  └─MachineDeployment/tkg-mgmt-docker-20210601125056-md-0
    └─Machine/tkg-mgmt-docker-20210601125056-md-0-5d895cbfd9-khj4s                              True                     24s
      └─MachineInfrastructure - DockerMachine/tkg-mgmt-docker-20210601125056-md-0-d544k


Providers:

  NAMESPACE                          NAME                   TYPE                    PROVIDERNAME  VERSION  WATCHNAMESPACE
  capd-system                        infrastructure-docker  InfrastructureProvider  docker        v0.3.10
  capi-kubeadm-bootstrap-system      bootstrap-kubeadm      BootstrapProvider       kubeadm       v0.3.14
  capi-kubeadm-control-plane-system  control-plane-kubeadm  ControlPlaneProvider    kubeadm       v0.3.14
  capi-system                        cluster-api            CoreProvider            cluster-api   v0.3.14
```
4. Capture the management cluster’s kubeconfig and take note of the command for accessing the cluster in the message, as you will use this for setting the context in the next step.

**Note** The below full command will be occurred on the last line from last step. Replacing <MGMT-CLUSTER-NAME> with your specific name.  

```
tanzu management-cluster kubeconfig get <MGMT-CLUSTER-NAME> --admin
```
Where <MGMT-CLUSTER-NAME> should be set to the name returned by tanzu management-cluster get.
For example, if your management cluster is called ‘mtce’, you will see a message similar to:
```
Credentials of cluster 'mtce' have been saved.
You can now access the cluster by running 'kubectl config use-context mtce-admin@mtce'
```
5. Set your kubectl context to the management cluster.
```
kubectl config use-context <MGMT-CLUSTER-NAME>-admin@<MGMT-CLUSTER-NAME>
```
6. Validate you can access the management cluster’s API server.
```
kubectl get nodes
```
You will see output similar to:
```
NAME                         STATUS   ROLES                  AGE   VERSION
guest-control-plane-tcjk2    Ready    control-plane,master   59m   v1.20.4+vmware.1
guest-md-0-f68799ffd-lpqsh   Ready    <none>                 59m   v1.20.4+vmware.1
```

#### 4.2.3 Deploy a Workload Cluster

1. Create your workload cluster.

```
tanzu cluster create <WORKLOAD-CLUSTER-NAME> --plan dev
```

2. Validate the cluster starts successfully.

```tanzu cluster list
```

3. Capture the workload cluster’s kubeconfig.
```
tanzu cluster kubeconfig get <WORKLOAD-CLUSTER-NAME> --admin
```

4. Set your kubectl context accordingly.

```
kubectl config use-context <WORKLOAD-CLUSTER-NAME>-admin@<WORKLOAD-CLUSTER-NAME>
```

5. Verify you can see pods in the cluster.

```
kubectl get pods --all-namespaces
```

The output will look similar to the following:

```
NAMESPACE     NAME                                                    READY   STATUS    RESTARTS   AGE
kube-system   antrea-agent-9d4db                                      2/2     Running   0          3m42s
kube-system   antrea-agent-vkgt4                                      2/2     Running   1          5m48s
kube-system   antrea-controller-5d594c5cc7-vn5gt                      1/1     Running   0          5m49s
kube-system   coredns-5d6f7c958-hs6vr                                 1/1     Running   0          5m49s
kube-system   coredns-5d6f7c958-xf6cl                                 1/1     Running   0          5m49s
kube-system   etcd-tce-guest-control-plane-b2wsf                      1/1     Running   0          5m56s
kube-system   kube-apiserver-tce-guest-control-plane-b2wsf            1/1     Running   0          5m56s
kube-system   kube-controller-manager-tce-guest-control-plane-b2wsf   1/1     Running   0          5m56s
kube-system   kube-proxy-9825q                                        1/1     Running   0          5m48s
kube-system   kube-proxy-wfktm                                        1/1     Running   0          3m42s
kube-system   kube-scheduler-tce-guest-control-plane-b2wsf            1/1     Running   0          5m56s
```
You now have local clusters running on Docker. The nodes can be seen by running the following command:

```
docker ps
```

The output will be similar to the following:

```
CONTAINER ID   IMAGE                                                         COMMAND                  CREATED             STATUS             PORTS                                  NAMES
33e4e422e102   projects.registry.vmware.com/tkg/kind/node:v1.20.4_vmware.1   "/usr/local/bin/entr…"   About an hour ago   Up About an hour                                          guest-md-0-f68799ffd-lpqsh
4ae2829ab6e1   projects.registry.vmware.com/tkg/kind/node:v1.20.4_vmware.1   "/usr/local/bin/entr…"   About an hour ago   Up About an hour   41637/tcp, 127.0.0.1:41637->6443/tcp   guest-control-plane-tcjk2
c0947823840b   kindest/haproxy:2.1.1-alpine                                  "/docker-entrypoint.…"   About an hour ago   Up About an hour   42385/tcp, 0.0.0.0:42385->6443/tcp     guest-lb
a2f156fe933d   projects.registry.vmware.com/tkg/kind/node:v1.20.4_vmware.1   "/usr/local/bin/entr…"   About an hour ago   Up About an hour                                          mgmt-md-0-b8689788f-tlv68
128bf25b9ae9   projects.registry.vmware.com/tkg/kind/node:v1.20.4_vmware.1   "/usr/local/bin/entr…"   About an hour ago   Up About an hour   40753/tcp, 127.0.0.1:40753->6443/tcp   mgmt-control-plane-9rdcq
e59ca95c14d7   kindest/haproxy:2.1.1-alpine                                  "/docker-entrypoint.…"   About an hour ago   Up About an hour   35621/tcp, 0.0.0.0:35621->6443/tcp     mgmt-lb
```
The above reflects 1 management cluster and 1 workload cluster, both featuring 1 control plane node and 1 worker node. Each cluster gets an haproxy container fronting the control plane node(s). This enables scaling the control plane into an HA configuration.


-----------

 ### References
1. Tanzu Documentation: https://tanzucommunityedition.io/docs/v0.12/
2. Docker Documentation: https://docs.docker.com/get-docker/
3. Kubenetes Documentation: https://kubernetes.io/docs/home/
