# bind 服务器的配置



## 基础知识

1. 域名

   www.hd.com

   主机名 域名 类型

   ​            www.google.com

   第二层 google 主机名 .com 域名

   第三层 www 主机名 .google.com 域名

   名字acmecompany通常是主机名，acmecompany.则是全域名，.是根域名服务器

2. DNS解析库

   |  类型 | 描述                       |
   | ----: | :------------------------- |
   |     A | Address地址 IPv4           |
   |  AAAA | Address地址 IPv6           |
   |    NS | Name Server域名服务器      |
   |   SOA | Start of Authority授权状态 |
   |    MX | Mail Exchanger邮件交换     |
   | CNAME | Canonical Name规范名       |
   |   PTR | Pointer指针                |

3. 顺序

   - 本地Hosts文件
   - 本地DNS缓存
   - 本地DNS服务器
   - 发起迭代查询

4. DNS端口号

   客户端向DNS请求使用UDP的53，DNS服务器间区域传送使用TCP的53



## CoreDNS安装

https://coredns.io/

``` bash
安装etcd
yum install etcd -y
systemctl enable --now etcd

安装coredns
wget ...
tar zxvf coredns_1.3.0_linux_amd64.tgz 
mv coredns /usr/bin 
mkdir /etc/coredns

安装dig
yum install -y bind-utils
```



## CoreDNS配置

- 创建配置文件/etc/coredns/Corefile

  ``` bash
  vim /etc/coredns/Corefile
  ```

- 编写配置

  ``` bash
  .:53 {
      etcd {
          stubzones
          path /coredns
          endpoint http://localhost:2379
  
          upstream 119.29.29.29:53 223.5.5.5:53 180.76.76.76:53
  
          fallthrough
      }
      prometheus
      cache 160
      loadbalance
      hosts {
          192.168.6.91 core.harbor.dc
          192.168.6.91 org-service.dc
          fallthrough
      }
      forward . 119.29.29.29:53 223.5.5.5:53 180.76.76.76:53
      log
  }
  ```

  