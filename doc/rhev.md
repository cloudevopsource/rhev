# 标准化安装

硬盘合规、安装RedHatEnterpriseLinux 7

1.3T   	制备：LVM

## 创建目录

```bash
/boot        size： 512MB

/boot/efi    size： 512MB

swap         size： 32GB    VG name ：rhev-vg

/            size： 100GB

/var         size： 100GB

/iso         size： 30GB

/vmtemp      size:  70GB

/vmdata1     size:  1008GB
```
NETWORK&HOST NAME :rhvm1  fzport.com   

网卡：enp61s0f1 

ipv4 manual   172.16.10.1

密码:Undead@666

**补充：bond0（为万兆网口）**        

```bahs
ens1f0               ensp60s0f0
```
 bond0 ：{                                } ovirt （为千兆口）
          ens2f0               ems5f0

确认主机是否满足虚拟化（bios开启虚拟化）

**grep -E 'svm|vmx' /proc/cpuinfo | grep nx #输出值满足**

## 向RedHat订阅管理器注册系统（注册可在官网订阅）

```bash
# subscription-manager register               #注册账号：fzgwjt
# subscription-manager list --available       #查看可订阅池，获取对应pollid
# subscription-manager attach --pool=pool_id  #注册对应订阅仓库池
# subscription-manager list --consumed        #查看系统当前附加的订阅
```

## 启用仓库包

```bash
# subscription-manager repos \
    --disable='*' \
    --enable=rhel-7-server-rpms \
    --enable=rhel-7-server-supplementary-rpms \
    --enable=rhel-7-server-rhv-4.3-manager-rpms \
    --enable=rhel-7-server-rhv-4-manager-tools-rpms \
    --enable=rhel-7-server-ansible-2-rpms \
    --enable=jb-eap-7.2-for-rhel-7-server-rpms \
    --enable=rhel-7-server-rhv-4-mgmt-agent-rpms \
    --enable=rhel-7-server-ansible-2-rpms
```

## 升级kernel

### 下载离线包kernel_4.19

```bash
172.16.10.1:/stage
```


### 删除旧内核
```bash
yum remove -y  kernel-devel*
yum remove -y  kernel-tools*
yum remove -y  kernel-header*
```


### 查看当前内核
```
rpm -qa | grep kernel
```

### 配置yum代理
```bash
vi /etc/yum.conf
proxy=http://172.16.130.106:7078
export http_proxy="http://172.16.130.106:7078"
export https_proxy="https://172.16.130.106:7078"
```


### 下载kernel内核包其他依赖包
```bash
yum install perl* 
```

### 安装/stage下kernenl4.19rpm包
```
rpm -ivh *.rpm     
```

### 设置默认启动项
```bash
grub2-set-default 0
```


### 更新grub.cfg
```
grub2-mkconfig -o /boot/grub2/grub.cfg
```

### 重启切换到新内核
```
reboot
```

### 检查系统存在内核
```
uname -r
rpm -qa | grep kernel
```

### 若存在旧版本，则删除旧版本
```
yum remove -y kernel-3.10.0*
```

### 升级软件包
```bash
yum update   
```

### 安装时间同步并配置

```bash
 yum install -y chrony#时间同步（172.16.102.199）
 systemctl enable chronyd
 systemctl start chronyd 
```

 vi /etc/chrony.conf  （添加以下两行，其他注释）

```bash
 server 172.16.102.199 iburst
 allow 172.16.102.199
```

### 检查时间同步

```bash
systemctl restart chronyd

# 查看时间同步状态
timedatectl status

# 开启网络时间同步
timedatectl set-ntp true

# 查看 ntp_servers
chronyc sources -v

# 查看 ntp_servers 状态
chronyc sourcestats -v

# 查看 ntp_servers 是否在线
chronyc activity -v

# 查看 ntp 详细信息
chronyc tracking -v

```

## 安装rhvm包和依赖项

```bash
yum install rhvm
```

运行engine-setup命令开始配置RedHat虚拟化管理器

```bash
engine-setup 
```

```bash
[ INFO  ] Stage: Initializing
[ INFO  ] Stage: Environment setup
          Configuration files: ['/etc/ovirt-engine-setup.conf.d/10-packaging-wsp.conf', '/etc/ovirt-engine-setup.conf.d/10-packaging.conf']
          Log file: /var/log/ovirt-engine/setup/ovirt-engine-setup-20200108192737-7hdehd.log
          Version: otopi-1.8.4 (otopi-1.8.4-1.el7ev)
[ INFO  ] Stage: Environment packages setup
[ INFO  ] Stage: Programs detection
[ INFO  ] Stage: Environment setup (late)
[ INFO  ] Stage: Environment customization
```

​         

          --== PRODUCT OPTIONS ==--
         
          Set up Cinderlib integration
          (Currently in tech preview)
          (Yes, No) [No]: 
          Configure Engine on this host (Yes, No) [Yes]: 
          Configure ovirt-provider-ovn (Yes, No) [Yes]: 
          Configure WebSocket Proxy on this host (Yes, No) [Yes]: 
         
          * Please note * : Data Warehouse is required for the engine.
          If you choose to not configure it on this host, you have to configure
          it on a remote host, and then configure the engine on this host so
          that it can access the database of the remote Data Warehouse host.
          Configure Data Warehouse on this host (Yes, No) [Yes]: 
          Configure Image I/O Proxy on this host (Yes, No) [Yes]: 
          Configure VM Console Proxy on this host (Yes, No) [Yes]: 
         
          --== PACKAGES ==--

[ INFO  ] Checking for product updates...
[ INFO  ] No product updates found
         
          --== NETWORK CONFIGURATION ==--
         
          Host fully qualified DNS name of this server [rhvh.fzport.com]: 
[WARNING] Failed to resolve rhvh.fzport.com using DNS, it can be resolved only locally
          Setup can automatically configure the firewall on this system.
          Note: automatic configuration of the firewall may overwrite current settings.
          NOTICE: iptables is deprecated and will be removed in future releases
          Do you want Setup to configure the firewall? (Yes, No) [Yes]: 
[ INFO  ] firewalld will be configured as firewall manager.
         
          --== DATABASE CONFIGURATION ==--
         
          Where is the DWH database located? (Local, Remote) [Local]: 
          Setup can configure the local postgresql server automatically for the DWH to run. This may conflict with existing applications.
          Would you like Setup to automatically configure postgresql and create DWH database, or prefer to perform that manually? (Automatic, Manual) [Automatic]: 
          Where is the Engine database located? (Local, Remote) [Local]: 
          Setup can configure the local postgresql server automatically for the engine to run. This may conflict with existing applications.
          Would you like Setup to automatically configure postgresql and create Engine database, or prefer to perform that manually? (Automatic, Manual) [Automatic]: 
         
          --== OVIRT ENGINE CONFIGURATION ==--
         
          Engine admin password: 
          Confirm engine admin password: 
[WARNING] Password is weak: The password is shorter than 8 characters
          Use weak password? (Yes, No) [No]: yes
          Application mode (Virt, Gluster, Both) [Both]: 
          Use default credentials (admin@internal) for ovirt-provider-ovn (Yes, No) [Yes]: 
         
          --== STORAGE CONFIGURATION ==--
         
          Default SAN wipe after delete (Yes, No) [No]: 
         
          --== PKI CONFIGURATION ==--
         
          Organization name for certificate [fzport.com]: 
         
          --== APACHE CONFIGURATION ==--
         
          Setup can configure the default page of the web server to present the application home page. This may conflict with existing applications.
          Do you wish to set the application as the default page of the web server? (Yes, No) [Yes]: 
          Setup can configure apache to use SSL using a certificate issued from the internal CA.
          Do you wish Setup to configure that, or prefer to perform that manually? (Automatic, Manual) [Automatic]: 
         
          --== SYSTEM CONFIGURATION ==--


​         
​          --== MISC CONFIGURATION ==--
​         
          Please choose Data Warehouse sampling scale:
          (1) Basic
          (2) Full
          (1, 2)[1]: 
         
          --== END OF CONFIGURATION ==--

[ INFO  ] Stage: Setup validation
         
          --== CONFIGURATION PREVIEW ==--
         
          Application mode                        : both
          Default SAN wipe after delete           : False
          Firewall manager                        : firewalld
          Update Firewall                         : True
          Host FQDN                               : rhvh.fzport.com
          Set up Cinderlib integration            : False
          Configure local Engine database         : True
          Set application as default page         : True
          Configure Apache SSL                    : True
          Engine database secured connection      : False
          Engine database user name               : engine
          Engine database name                    : engine
          Engine database host                    : localhost
          Engine database port                    : 5432
          Engine database host name validation    : False
          Engine installation                     : True
          PKI organization                        : fzport.com
          Set up ovirt-provider-ovn               : True
          Configure WebSocket Proxy               : True
          DWH installation                        : True
          DWH database host                       : localhost
          DWH database port                       : 5432
          Configure local DWH database            : True
          Configure Image I/O Proxy               : True
          Configure VMConsole Proxy               : True
         
          Please confirm installation settings (OK, Cancel) [OK]: 
[ INFO  ] Stage: Transaction setup
[ INFO  ] Stopping engine service
[ INFO  ] Stopping ovirt-fence-kdump-listener service
[ INFO  ] Stopping dwh service
[ INFO  ] Stopping Image I/O Proxy service
[ INFO  ] Stopping vmconsole-proxy service
[ INFO  ] Stopping websocket-proxy service
[ INFO  ] Stage: Misc configuration (early)
[ INFO  ] Stage: Package installation
[ INFO  ] Stage: Misc configuration
[ INFO  ] Upgrading CA
[ INFO  ] Creating PostgreSQL 'engine' database
[ INFO  ] Configuring PostgreSQL
[ INFO  ] Creating PostgreSQL 'ovirt_engine_history' database
[ INFO  ] Configuring PostgreSQL
[ INFO  ] Creating CA
[ INFO  ] Creating/refreshing DWH database schema
[ INFO  ] Configuring Image I/O Proxy
[ INFO  ] Setting up ovirt-vmconsole proxy helper PKI artifacts
[ INFO  ] Setting up ovirt-vmconsole SSH PKI artifacts
[ INFO  ] Configuring WebSocket Proxy
[ INFO  ] Creating/refreshing Engine database schema
[ INFO  ] Creating/refreshing Engine 'internal' domain database schema
[ INFO  ] Creating default mac pool range
[ INFO  ] Adding default OVN provider to database
[ INFO  ] Adding OVN provider secret to database
[ INFO  ] Setting a password for internal user admin
[ INFO  ] Generating post install configuration file '/etc/ovirt-engine-setup.conf.d/20-setup-ovirt-post.conf'
[ INFO  ] Stage: Transaction commit
[ INFO  ] Stage: Closing up
[ INFO  ] Starting engine service
[ INFO  ] Starting dwh service
[ INFO  ] Restarting ovirt-vmconsole proxy service
         
          --== SUMMARY ==--

[ INFO  ] Restarting httpd
          Please use the user 'admin@internal' and password specified in order to login
          Web access is enabled at:
              http://rhvh.fzport.com:80/ovirt-engine
              https://rhvh.fzport.com:443/ovirt-engine
          Internal CA 2C:A1:20:6A:80:42:CD:89:64:5A:E2:9C:2D:61:DD:A3:9F:93:63:AC
          SSH fingerprint: SHA256:doneFsGWmnk5v1N4ljvPvjmu6yngUhdnF4V+X4cZdlI
         
          --== END OF SUMMARY ==--

[ INFO  ] Stage: Clean up
          Log file is located at /var/log/ovirt-engine/setup/ovirt-engine-setup-20200108192737-7hdehd.log
[ INFO  ] Generating answer file '/var/lib/ovirt-engine/setup/answers/20200108193511-setup.conf'
[ INFO  ] Stage: Pre-termination
[ INFO  ] Stage: Termination
[ INFO  ] Execution of setup completed successfully

#使用Web浏览器访问管理门户
http://rhvh.fzport.com:80/ovirt-engine  

#使用备用主机名或IP地址访问管理门户 
vi /etc/ovirt-engine/engine.conf.d/99-custom-sso-setup.conf

SSO_ALTERNATE_ENGINE_FQDNS="alias1.example.com alias2.example.com" 
#备用主机名的列表需要用空格分隔。IP地址添加到列表中，但不建议使用IP地址而不是DNS解析主机名。

## 安装仪表板包

```bash
yum install cockpit-ovirt-dashboard
```

启用并启动cockpit.socket服务：
```bash
# systemctl enable cockpit.socket

# systemctl start cockpit.socket
```

检查驾驶舱是否是防火墙中的有效服务：
```bash
# firewall-cmd --list-services

# firewall-cmd --permanent --add-service=cockpit
```

登录到驾驶舱网络界面https://HostFQDNorIP:9090.

## 虚拟化化管理器配置集群、数据中心、主机、存储域

### 配置集群、数据中心

登陆虚拟化管路平台
http://rhvh.fzport.com:80/ovirt-engine

点击“计算“—”数据中心“—“新建”—“配置/本地”确认—“配置集群/确认”—”配置主机/确认“，点击“计算“—”主机“查看安装状态；
点击”事件“查看安装日志是否存在报错信息

### 添加主机后建立存储域并激活

NFS：配置ISO存储
创建组kvm:

```bash
groupadd kvm -g 36
```

创建用户vdsm在小组中kvm:
```bash
useradd vdsm -u 36 -g 36
```

将导出目录的所有权设置为36：36，这将vdsm:kvm所有权：
新建目录：mkdir -p /data/iso

```bash
chown -R 36:36 /data/iso
```

更改目录的模式，以便将读和写访问权限授予所有者，并将读和执行访问权限授予组和其他用户：
```bash
chmod 0755 /data/iso
```

本地存储：配置数据存储data和iso
#红帽企业Linux主机本地存储的准备
在主机上，创建用于本地存储的目录：
```bash
mkdir -p /data/images
```

确保该目录具有允许对vdsm用户(UID 36)和KVM组(GID 36) ：
```bash
# chown 36:36 -R  /data /data/images

# chmod 0755 -R /data /data/images
```

#红帽虚拟化主机本地存储的准备
RedHat建议在逻辑卷上创建本地存储，如下所示：
创建一个本地存储目录：
```bash
mkdir /data

lvcreate -L $SIZE rhvh -n data

mkfs.ext4 /dev/mapper/rhvh-data

echo "/dev/mapper/rhvh-data /data ext4 defaults,discard 1 2" >> /etc/fstab

mount /data
```

#装入新的本地存储，然后修改权限和所有权：
```bash
mount -a

chown 36:36 /data /rhvh-data

chmod 0755 /data /rhvh-data
```

添加虚拟化管理平台置激活数据中心、存储域/ISO
点击“存储”—“域”—新建域—“配置存储域”—“名称/数据中心（rhvh）/域功能（数据）/存储类型（本地）/使用主机（172.16.10.1）/路径（/data）“
点击“存储”—“域”—新建域—“配置存储域”—“名称/数据中心（rhvh）/域功能（ISO）/存储类型（本地或NFS）使用主机（172.16.10.1）/路径（/data/images）“

## 配置网络、虚拟化

### 配置网络

### 添加虚拟机

### 安装客户端安装控制台支持插件

### 红帽企业Linux上安装远程查看器

```bash
yum install virt-viewer
```

在Windows上安装远程查看器
打开虚拟化管理平台
根据系统架构32/64下载控制台客户端资源
下载安装VirtViewer和usbdk；

# 问题记录

**engine自启动引擎虚拟机 无法启动维护**

引擎虚拟机启动报错The hosted engine configuration has not been retrieved from shared storage. Please ensure that ovirt
问题在共享存储
以glusterfs为例,三个多副本节点剩下一个节点可用

**解决步骤如下**

1、尝试手动挂载glusterfs

```bash
mount -t glusterfs ovm6stor:/gv_data /mnt/test
```

2、无法挂在因为glusterfs发生故障，处理顺序:停止卷\先剔出故障节点的brick副本\在剔出故障peer节点\再次启动卷

```bash
gluster volume stop gv_data
gluster volume stop gv_iso
gluster volume info
gluster volume remove-brick gv_data  replica 2 ovm4stor:/gluster_bricks/data force
gluster volume remove-brick gv_data  replica 1 ovm5stor:/gluster_bricks/data force
gluster volume info
gluster volume remove-brick gv_iso  replica 2 ovm5stor:/gluster_bricks/iso force
gluster volume remove-brick gv_iso  replica 1 ovm4stor:/gluster_bricks/iso force
gluster volume info
gluster peer detach 100.100.100.5 force
gluster peer detach 100.100.100.4 force
gluster volume start gv_data
gluster volume start gv_iso
```

3、重启代理systemctl restart ovirt-ha-agent,还是不行
4、检查engine 配置文件

```
vi /etc/ovirt-hosted-engine/hosted-engine.conf
```

```
#修改
storage=ovm6stor:/gv_data
mnt_options=backup-volfile-servers=ovm4stor:ovm5stor
```


5、重启代理systemctl restart ovirt-ha-agent

hosted-engine  --connect-storage
hosted-engine --console
hosted-engine --vm-status
hosted-engine --vm-start
