# Kubernetes文档

## 目录

1. [Gitlab CI/CD](0-Gitlab_CICD.md)
2. [Kubernetes](1-Kubernetes.md)
3. [存储](2-存储.md)
4. [网络](3-网络.md)
5. [添加节点](9-配置集群_全新机器.md)

### 排水

```bash
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
kubectl uncordon <node>
```

[修改默认路径](https://www.cnblogs.com/datasyman/p/7307085.html)

```bash
sudo systemctl stop docker
sudo mkdir /etc/systemd/system/docker.service.d
sudo touch /etc/systemd/system/docker.service.d/docker.conf

# sudo vi /etc/systemd/system/docker.service.d/docker.conf
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd --graph="/data/docker" --storage-driver=devicemapper
```

提权进 docker

```bash
docker exec -it --user=0 --privileged 378ab6ac63b4 bash
```

alpine 阿里源

```bash
echo "https://mirrors.aliyun.com/alpine/v3.11/main" > /etc/apk/repositories ; echo "https://mirrors.aliyun.com/alpine/v3.11/community" >> /etc/apk/repositories
```

echo 'deb [Index of /debian/](http://mirrors.ustc.edu.cn/debian) stable main contrib non-free' >>/etc/apt/sources.list
 echo 'deb http://mirrors.ustc.edu.cn/debian stable-updates main contrib non-free' >>/etc/apt/sources.list
