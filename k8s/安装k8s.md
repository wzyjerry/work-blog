# 安装k8s

## 安装docker

1. 换源
   
   ```bash
   curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
   ```

2. 安装
   
   ```bash
   curl -sSL https://get.daocloud.io/docker | sh
   systemctl enable --now docker
   ```

3. 换源
   
   ```bash
   vim /etc/docker/daemon.json
   {
     "registry-mirrors": ["http://hub-mirror.c.163.com"],
     "insecure-registries": ["harbor.base.dc"],
     "exec-opts": ["native.cgroupdriver=systemd"],
     "data-root": "/data/docker"
   }
   systemctl restart docker
   ```

4. 修改代理（如果需要）
   
   ```bash
   vim /etc/systemd/system/docker.service.d/http-proxy.conf
   [Service]
   Environment="HTTPS_PROXY=http://10.0.0.1:3128" "HTTP_PROXY=http://10.0.0.1:3128"
   ```

## 安装kubelet

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

1. 换源：
   
   ```bash
   cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
   [kubernetes]
   name=Kubernetes
   baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-\$basearch
   enabled=1
   gpgcheck=1
   repo_gpgcheck=1
   gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
   exclude=kubelet kubeadm kubectl
   EOF
   ```
   
   ```bash
   sudo setenforce 0
   sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
   
   yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
   yum install -y kubelet kubectl kubeadm
   systemctl enable --now kubelet
   ```

2. 镜像预载（注意更改版本号）
   
    https://blog.51cto.com/3241766/2405624
   
   ```bash
   #!/bin/bash
   url=registry.cn-hangzhou.aliyuncs.com/google_containers
   version=v1.22.1
   images=(`kubeadm config images list --kubernetes-version=$version|awk -F '/' '{print $2}'`)
   for imagename in ${images[@]} ; do
     docker pull $url/$imagename
     docker tag $url/$imagename k8s.gcr.io/$imagename
     docker rmi -f $url/$imagename
   done
   ```

3. 初始化
   
   ```bash
   yum install bash-completion -y
   echo "source <(kubectl completion bash)" >> ~/.bash_profile
   source .bash_profile
   
   kubeadm init --apiserver-advertise-address 10.0.0.15 --pod-network-cidr=10.244.0.0/16
   echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile
   source ~/.bash_profile
   
   kubeadm init \
   --apiserver-advertise-address=10.0.0.4 \
   --pod-network-cidr=10.244.0.0/16
   ```

4. Node加入集群
   
   ```bash
   kubeadm join 10.0.0.15:6443 --token xb9vcz.ke42d283zqpt96w3 --discovery-token-ca-cert-hash sha256:2d44b2426f3ac68ecfda0fe26b1330cee47107a72863e58ebfeabe9108d3b71b
   ```

5. 切换DNS
   
   ```bash
   vim /etc/resolv.conf
   nameserver 180.76.76.76
   ```

6. 安装pod网络
   
   ```bash
   kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
   ```

7. 自动补全
   
   ```bash
   yum install -y bash-completion
   source /usr/share/bash-completion/bash_completion
   source <(kubectl completion bash)
   echo "source <(kubectl completion bash)" >> ~/.bashrc
   ```
   
    #添加kubectl的k别名
    vim   ~/.bashrc 
    alias k='kubectl'
   
    #tab命令只在使用完整的kubectl 命令起作用，使用别名k 时不起作用，修补：
    source <( kubectl completion bash | sed 's/kubectl/k/g' )  #写入 .bashrc 
   
   ```
   
   ```

```sh
kubectl create secret docker-registry annotation --docker-server=docker.aminer.cn --docker-username=annotation --docker-password=2W5s47X62t8FvD7z7qxw -n organization
```
