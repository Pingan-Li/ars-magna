# Samba
## 1. 安装Samba
使用dnf安装Samba之后还需要用systemctl其中服务器，以及添加防火墙规则
```shell
sudo dnf install samba
sudo systemctl enable smb --now
firewall-cmd --get-active-zones
sudo firewall-cmd --permanent --zone=FedoraWorkstation --add-service=samba
sudo firewall-cmd --reload
```

## 2. 添加Samba用户
安装完之后还需要添加用户，因为Samba不使用操作系统的用户进行授权，
所以Samba用户和系统用户是独立的。
```shell
sudo smbpasswd -a jane
```

## 3. 创建共享目录
Fedora不仅默认开启了防火墙，而且还默认开启了SELiunx，因此在创建共享目录后，需要添加SELiunx的规则。
```shell
mkdir /home/jane/share
sudo semanage fcontext --add --type "samba_share_t" "/home/jane/share(/.*)?"
sudo restorecon -R ~/share
```
注意：这里实际上还是没有权限，配置的时候遇到了这个问题无法解决，于是只能改成：
```shell
sudo semanage fcontext --add --type "samba_share_t" "/home/jane(/.*)?"
```
## 4. 配置Samba
```text
[share]
        comment = My Share
        path = /home/jane/share
        writeable = yes
        browseable = yes
        public = yes
        create mask = 0644
        directory mask = 0755
        write list = user
```
## 5. 重启Samba服务
```shell
sudo systemctl restart smb
```