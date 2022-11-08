# Linux

## 系统管理

### yum

### systemctl

**systemctl命令**是系统服务管理器指令，它实际上将 service 和 chkconfig 这两个命令组合到一起。

- `systemctl enable httpd.service`  使某服务自动启动
- `systemctl disable httpd.service`  使某服务不自动启动
- `systemctl status httpd.service`  服务详细信息
- `systemctl is-active httpd.service`  仅显示是否 Active
- `systemctl list-units --type=service`  显示所有已启动的服务
- `systemctl start httpd.service`  启动某服务
- `systemctl stop httpd.service`  停止某服务
- `systemctl restart httpd.service`  重启某服务
- `systemctl cat httpd.service`  查看配置文件的内容

### vim

## 网络管理

### 推荐阅读

[Linux命令](https://man.linuxde.net/)