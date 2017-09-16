# 如何在Ubuntu上安装Keystone
最近在学习Keystone，打算在家里的笔记本上尝试安装Keystone，方便以后上手操作。参考材料来自于OpenStack官网的安装指南，但实际操作中也出现部分官网指南没有包含到的地方，特意整理出来。
参考文档: [OpenStack安装指南](https://docs.openstack.org/newton/install-guide-ubuntu/index.html),[KeyStone安装指南](https://docs.openstack.org/keystone/latest/install/index-ubuntu.html)
    
## 安装版本
    OpenStack Ocata
    
## 环境
    硬件： 笔记本电脑，物理机，CPU 2核1.87G， 内存 2G
    操作系统： Ubuntu 16.04 TLS
    
## 前置部分安装
使用root用户安装

### 启动OpenStack Repository
将OpenStack Repository添加到Ubuntu的仓库中才能用apt install命令来安装keystone和OpenStack Client。

    # apt install software-properties-common
    # add-apt-repository cloud-archive:ocata
    # apt update && apt dist-upgrade
    
这一步安装完成后需要重启操作系统。

### 安装OpenStack Client
    # apt install python-openstackclient
    
### 安装Mysql

这里安装的Mysql是MariaDB，即Mysql的社区化版本。

1. 安装MariaDB软件包

    > \# apt install mariadb-server python-pymysql

2. 配置Mysql
    > \# vim /etc/mysql/mariadb.conf.d/99-openstack.cnf
    
    在文件中创建下面这一节，并把bind-address替换成本机真实ＩＰ。
    
'''json
    \[mysqld\]
    bind-address = 10.0.0.11
    default-storage-engine = innodb
    innodb_file_per_table
    max_connections = 4096
    collation-server = utf8_general_ci
    character-set-server = utf8
'''


