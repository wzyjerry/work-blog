# 搭建FTP

https://blog.csdn.net/sinat_30802291/article/details/81706152

1. 安装vsftpd

   ``` bash
   yum -y install vsftpd
   ```

2. 启动服务

   ``` bash
   systemctl enable --now vsftpd
   ```

3. 修改配置

   ``` bash
   vim /etc/vsftpd/vsftpd.conf
   
   ftpd_banner=Welcome to AMiner FTP service.
   anonymous_enable=NO
   chroot_local_user=YES
   chroot_list_enable=YES
   chroot_list_file=/etc/vsftpd/chroot_list
   guest_enable=YES
   guest_username=ftpuser
   user_config_dir=/etc/vsftpd/vuser_conf
   allow_writeable_chroot=YES
   pasv_min_port=6000
   pasv_max_port=6010
   ```

4. 创建宿主用户

   ``` bash
   cd /home
   mkdir vsftpd
   useradd -g root -M -d /home/vsftpd -s /sbin/nologin ftpuser
   passwd ftpuser
   chown -R ftpuser.root /home/vsftpd
   ```

5. 创建虚拟用户

   ``` bash
   vim /etc/vsftpd/vuser_passwd
   
   username
   password
   ...
   ```

6. 生成数据库

   ``` bash
   db_load -T -t hash -f /etc/vsftpd/vuser_passwd /etc/vsftpd/vuser_passwd.db
   chmod 600 /etc/vsftpd/vuser_passwd.db
   ```

7. 编辑pam认证文件

   ``` bash
   vim /etc/pam.d/vsftpd
   
   注释掉其他
   auth required /lib64/security/pam_userdb.so db=/etc/vsftpd/vuser_passwd
   account required /lib64/security/pam_userdb.so db=/etc/vsftpd/vuser_passwd
   ```

8. 创建虚拟账户根目录

   ``` bash
   /home/vsftpd/username   (上传根目录，可软链到数据目录)
   ```

9. 创建虚拟账户配置目录

   ``` bash
   cd /etc/vsftpd/
   mkdir vuser_conf
   cd vuser_conf
   ```

10. 创建虚拟账户配置

    ``` bash
    vim username
    
    local_root=/home/vsftpd/ username
    write_enable=YES
    anon_umask=022
    anon_world_readable_only=NO
    anon_upload_enable=YES
    anon_mkdir_write_enable=YES
    anon_other_write_enable=YES
    ```

11. 创建chroot_list

    ``` bash
    vim chroot_list
    
    username
    ...
    ```

12. 重启vsftp服务

    ``` bash
    systemctl restart vsftpd
    ```
    
13. 创建软连接

    ``` bash
    ln -s /data/ftp/tmp-user /home/vsftpd/tmp-user
    ```

    


备注：

- FTP分为主动模式和被动模式，主动模式使用20作为数据连入端口，被动模式使用pasv_min_port到pasv_max_port作为数据连出端口。防火墙开启20、21、6000～6010