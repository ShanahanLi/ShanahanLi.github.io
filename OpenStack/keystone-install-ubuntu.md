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
    
'''

    [mysqld]
    bind-address = 10.0.0.11
    default-storage-engine = innodb
    innodb_file_per_table
    max_connections = 4096
    collation-server = utf8_general_ci
    character-set-server = utf8
'''

3. 重启ＭｙＳＱＬ，并进行安全设置

    > \# service mysql restart
    
    安全设置中需要设置数据库root密码设置
    > \# mysql_secure_installation

## 安装Keystone
### 创建keystone数据库实例

1. 以root用户身份连接Mysql数据库

    > \# mysql

注意，如果访问失败，可能Mysql服务没有启动，可通过下面命令重启
    
    # service mysql start

2. 创建数据库实例
    > MariaDB [(none)]> CREATE DATABASE keystone;
    
3. 给keystone数据库实例授权

    > MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'KEYSTONE_DBPASS';
    
    > MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'KEYSTONE_DBPASS';

注意: 'KEYSTONE_DBPASS'需要替换成合适的密码。

4. 退出Ｍｙｓｑｌ客户端

### 安装ｋｅｙｓｔｏｎｅ软件包
Keystone运行在Apache的容器中，并且以来Apache的wsgi模块，Ｋeystone默认监听５０００和３５３５７两个端口。

1. 安装keystone和Apache2软件包
    
    \# apt install keystone apache2 libapache2-mod-wsgi
  
### 配置keystone
１. 打开keystone.conf

    # vim /etc/keystone/keystone.conf

２. 修改database配置
    搜索\[database，将默认的配置注释，替换成下面的内容
    
    [database]  
    connection = mysql+pymysql://keystone:KEYSTONE_DBPASS@controller/keystone
    
注意controller需要替换成本机的ＩＰ地址。

３. 修改token生成方式
　　搜索\[token，将默认的配置注释，社区的安装指南是要求用fernet token，但是我需要用到pki token，所以设置为pki
  
    [token]
    provider = pki
    
### 初始化keystone


这个过程中，keystone-manage工具会往keystone数据库中插入初始化数据

    # su -s /bin/sh -c "keystone-manage db_sync" keystone
    
执行完成后，可以进入Ｍｙｓｑｌ查看keystone库里的表已经创建出来。

    # mysql
    MariaDB [(none)]> use keystone;
    MariaDB [keystone]> show tables;
    
+------------------------+
| Tables_in_keystone     |
+------------------------+
| access_token           |
| assignment             |
| config_register        |
| consumer               |
| credential             |
| domain                 |
| endpoint               |
| endpoint_group         |
| federated_user         |
| federation_protocol    |
| group                  |
| id_mapping             |
| identity_provider      |
| idp_remote_ids         |
| implied_role           |
| local_user             |
| mapping                |
| migrate_version        |
| password               |
| policy                 |
| policy_association     |
| project                |
| project_endpoint       |
| project_endpoint_group |
| region                 |
| request_token          |
| revocation_event       |
| role                   |
| sensitive_config       |
| service                |
| service_provider       |
| token                  |
| trust                  |
| trust_role             |
| user                   |
| user_group_membership  |
| whitelisted_config     |
+------------------------+

然后往Ｋｅｙｓｔｏｎｅ中插入初始化数据
 
    # keystone-manage bootstrap --bootstrap-password ADMIN_PASS \
    --bootstrap-admin-url http://controller:35357/v3/ \
    --bootstrap-internal-url http://controller:5000/v3/ \
    --bootstrap-public-url http://controller:5000/v3/ \
    --bootstrap-region-id RegionOne
    
注意，命令中的controller需要替换成本机ＩＰ地址，　ＡＤＭＩＮ_PASS需要替换成自定义的密码。
bootstrap过程中往keystone的表中插入了一些数据，例如Default domain, admin project。
  
## 配置Apache HTTP Server
在/etc/apache2/apache2.conf文件中配置ServerName，可以增加在文件最底部。

    ServerName controller

注意，如果apache2.conf配置文件不存在，说明Apache2还没有安装成功，请确认安装keystone软件包时apache2软件是否同时安装，如果没有安装，请安装Apache2及其wsgi模块。

    # apt install apache2 libapache2-mod-wsgi

然后重启Apache
    
    # service apache2 restart

至此，keystone已经安装完成，恭喜！

## 测试
keystone安装完成后，可以使用OpenStack Client调用，测试keystone是否正常运行。
调用OpenStack Client前，需要先设置环境变量。

在本地创建一个rc文件，使用OpenStack Client前将rc加载到环境变量中。 此处可以用非ｒｏｏｔ用户执行。

    $ vim ~/openstack-cli.rc
    
输入下面内容，其中controller需要替换成本机ＩＰ地址，　ＡＤＭＩＮ_PASS需要替换成自定义的密码。

    export OS_USERNAME=admin
    export OS_PASSWORD=ADMIN_PASS
    export OS_PROJECT_NAME=admin
    export OS_USER_DOMAIN_NAME=Default
    export OS_PROJECT_DOMAIN_NAME=Default
    export OS_AUTH_URL=http://controller:35357/v3
    export OS_IDENTITY_API_VERSION=3

    $ source openstack-cli.rc
    $ openstack token issue
    
返回token，则说明Ｋｅｙｓｔｏｎｅ可以正常运行！

  
