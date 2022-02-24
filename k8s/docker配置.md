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
