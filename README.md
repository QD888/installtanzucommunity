- 下载 Ubuntu 21.04: 
  https://old-releases.ubuntu.com/releases/21.04/ubuntu-21.04-live-server-amd64.iso
- 安装VMware Workstations，VM的需求如下
  |Resources   | Size/Version|
  | ------------|-------------|
  | OS          | Ubuntu V21.04 64 bit|
  | vCPU        | 4|
  | Memeory     | 8 GB |
  | Disk        | 50 GB|
- VM OS的基本配置和工具
  ```
  sudo apt update
  sudo apt install net-tools 
  sudo apt install curl wget
  ```
- IP地址配置所在位置
  ```
  cd /etc/netplan/
  sudo vi 01-network-manager-all.yaml
  sudo netplan apply
  ```
- 安装 kubectl
  ```
  curl -LO https://dl.k8s.io/release/v1.20.1/bin/linux/amd64/kubectl
  sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
  ```
- 安装 Docker
  ```
  sudo apt-get update
  sudo apt-get install ca-certificates curl gnupg lsb-release
  sudo mkdir -p /etc/apt/keyrings
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  sudo apt-get update
  sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
  ```
- 配置运行Docker的用户
  ```
  sudo groupadd docker
  sudo usermod -aG docker $USER
  newgrp docker
  exit
  sudo systemctl enable docker.service
  sudo systemctl enable containerd.service
  docker pull kindest/haproxy:v20210715-a6da3463
  ```
- Install Tanzu CLI 
  ```
  wget https://github.com/vmware-tanzu/community-edition/releases/download/v0.12.0/tce-linux-amd64-v0.12.0.tar.gz
  tar xzvf tce-linux-amd64-v0.12.0.tar.gz
  cd tce-linux-amd64-v0.12.0
  ./install.sh
  ```
- 启动Tanzu GUI
  ```
  # 改成自己的IP，ip addr 查看。如果要修改,配置文件在/etc/netplan/
  sudo apt-get install xdg-utils
  tanzu management-cluster create --ui -b 192.168.115.210:8080
  
  ```
- 浏览器访问http://<你自己VM的IP>:8080，选docker，next，起一个名字，next，点击Deploy
  这一步需要no split VPN网络，部署过程中会下载和上传大量的数据，要耐心等待
  如果Docker集群部署失败，断开 no splite VPN，重复上一步，换一个名字，比如manage
- 部署完成后，体验Tanzu
  ```
  tanzu management-cluster get
  tanzu management-cluster kubeconfig get tanzu --admin
  #注意：将下面管理集群名字manage替换为自己起的名字
  kubectl config use-context manage-admin@manage
  kubectl get node
  #创建工作负载集群，名字为tkg-workload
  tanzu cluster create tkg-workload --plan dev
  tanzu cluster list
  tanzu cluster  kubeconfig get tkg-workload --admin
  kubectl config get-contexts
  kubectl config use-context tkg-workload-admin@tkg-workload
  kubectl get node
  kubectl get pod --all-namespaces
  ```
- VM 休眠，避免下次用的时候IP发生变化，如果没有改成静态IP设置
- 参考文章
https://github.com/leo-cao/installtanzucommunity/blob/main/Tanzu%20Community%20Installation%20Guide.md
https://www.cnblogs.com/scofield666/p/15429376.html
