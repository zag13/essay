# VMware

## 配置

### 配置静态IP

> [VMware Fusion CentOS7配置静态IP](https://www.cnblogs.com/itbsl/p/10998696.html)

### 配置静态IP并允许上网

> [配置静态IP并允许上网](https://www.dearcloud.cn/archives/365)

1. 打开【系统偏好设置】-【网络】- 选中【Wi-Fi】项（如果您是WIFI上网请选择此项）- 点右侧【高级】

   选择【TCP/IP】选项卡，记录好【子网掩码】、【路由器】地址、DNS选项卡下的DNS服务器地址（如果DNS服务器地址没有配置，也可以给配置个8.8.8.8或者114.114.114.114）

   比如：

   路由器地址：192.168.0.1

   子网掩码：255.255.255.0

   DNS：114.114.114.114 （如果mac本上为空，也可以配置为8.8.8.8或者114.114.114.114）

2. 打开Vmware虚拟机，本文以Centos7为例：

    1. 在Centos7虚拟机关机状态下，打开Centos这台虚拟机的Vmware设置页面，选择【网络适配器】那一项。
    2. 在网络适配器中，左侧是当前选择的网络类型，您需切换到【Wi-Fi】那一项，方法是点击Wi-Fi旁边的单选按钮即可。

3. 打开Vmware虚拟机，并开启Centos7，并登陆root。

4. 执行： vi /etc/sysconfig/network-scripts/ifcfg-ens33 删除原配置，并修改为下列配置。

   ```shell
   TYPE="Ethernet"
   BOOTPROTO="static"
   DEFROUTE="yes"
   IPV4_FAILURE_FATAL="no"
   NAME="ens33"
   UUID="69ee63ff-e105-4786-935f-3b29e3a04a4e"
   DEVICE="ens33"
   ONBOOT="yes"
   IPADDR="192.168.0.10"
   NETMASK="255.255.255.0"
   GATEWAY="192.168.0.1"
   DNS1="114.114.114.114"
   IPV6_PEERDNS="yes"
   IPV6_PEEROUTES="yes"
   IPV6_PRIVACY="no"
   ```

   注意：IP地址可以新写一个地址，地址段要和您的MAC本的Wifi连接的路由器在同一网段。然后Gateway配置成您Mac本显示的【路由器地址】，NetMask配置为您的子网掩码。另外，要删除多余的配置，否则有可能上不了网。

5. 重启网卡

   ```shell
   systemctl restart network
   ```

6. 检查是否能上网

   ```shell
   ping www.baidu.com
   ```

7. 使用Macbook的SSH连接虚拟机

   ```shell
   ssh root@192.168.0.10
   ```

   





