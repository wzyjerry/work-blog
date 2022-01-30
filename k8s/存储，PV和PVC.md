# 存储，PV和PVC



## 创建nfs服务器用于提供存储

1. 安装nfs服务端

   ```bash
   yum install nfs-utils -y
   ```

2. 创建挂载点

   ``` bash
   mkdir -p /data/nfs
   ```

3. 编辑/etc/exports文件

   ``` bash
   vim /etc/exports
   /data/nfs 192.168.6.0/24(rw,no_root_squash)
   ```

4. 开启nfs服务和rpc服务

   ``` bash
   systemctl enable --now nfs
   systemctl enable --now rpcbind
   ```

5. 检测服务状态

   ``` bash
   showmount -e
   ```



## 安装存储类

1. 实例化一个默认存储类nfs-client

   ``` bash
   helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
   docker.io/dyrnq/nfs-subdir-external-provisioner:v4.0.2
   helm install nfs nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=192.168.6.147 --set nfs.path=/data/nfs --set storageClass.defaultClass=true
   ```
   
   



