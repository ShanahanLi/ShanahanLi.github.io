# OpenStack Client Quick Start

## 安装OpenStack Client
虽然OpenStack社区已经为很多操作系统提供来安装源，例如Ubuntu上可以直接用下面的命令安装：

    $ sudo apt-get install openstack-client
但是我建议通过pip安装，并且利用virtualenv，“绿色”安装OpenStack Client。
第一步：创建openstack client虚拟环境，并进入。

    $ virtualenv --no-site-packages --always-copy openstackclient-env
    $ cd openstackclient-env/
    $ source ./bin/activate
source执行后，所有命令前自动带上“(openstackclient-env)”
第二步：pip安装

    (openstackclient-env)$ pip install python-openstackclient
通过屏幕输出可以看到安装完成，openstack命令行安装在bin目录，在虚拟环境中可以直接执行。

第三步：查看帮助
执行--help查看全局配置和所有支持的命令行列表。

    (openstackclient-env)$ openstack --help
help命令可以获取具体命令的详细帮助。例如:

    (openstackclient-env)$ openstack help user create

## 源码安装OpenStack Client
如果你想要修改OpenStack Client源码来测试或者发布自己的命令，进入源码目录，执行：

    (openstackclient-env)$ python setup.py install
 或者
 
    (openstackclient-env)$ pip instal -e .
区别在于setup.py仅安装源码，不安装依赖，而pip instal可以做到。

## 使用OpenStack Client
OpenStack Client命令行有共同的格式：

    openstack [<global-options>] <object-1> <action> [<object-2>] [<command-arguments>]
global-options对所有命令行生效，例如认证的选项：    
    
    --os-auth-url <url>
    --os-identity-api-version 3
    --os-project-name <project-name>
    --os-project-domain-name <project-domain-name>
    --os-username <username>
    --os-user-domain-name <user-domain-name>
    [--os-password <password>]
从--help也可以看到，OpenStack Client有一大堆全局选项，但如果每次执行命令都带上这么多选项就太麻烦了，好在通过环境变量可以替换掉，不过如果两者同时存在，命令行全局选项优先。环境变量的名字就是全局选项前面的“--”去掉，然后中间的“-”替换为“_”，例如，全局选项--os-username可以被环境变量OS_USERNAME替换。

### 密码认证
创建一个文本文件，例如openstack_password.rc，然后输入下面内容：

    export OS_AUTH_URL=<url-to-openstack-identity>
    export OS_IDENTITY_API_VERSION=3
    export OS_PROJECT_NAME=<project-name>
    export OS_PROJECT_DOMAIN_NAME=<project-domain-name>
    export OS_USERNAME=<username>
    export OS_USER_DOMAIN_NAME=<user-domain-name>
    export OS_PASSWORD=<password>  # (plain text, optional)
密码可选，如果不输入密码，命令行会通过交互式方式提示输入。执行

    (openstackclient-env)# source ~/openstack_password.rc
    (openstackclient-env)# openstack token issue
    (openstackclient-env)# openstack image list --public
屏幕输出token，说明认证通过，可以正常访问。通过命令行查询所有公共镜像，也成功返回结果。

### SAML联邦认证
创建一个文本文件，例如openstack_saml2.rc，然后输入下面内容：

    export OS_IDENTITY_API_VERSION=3
    export OS_AUTH_TYPE=v3samlpassword
    export OS_AUTH_URL=https://iam.cn-north-1.myhwclouds.com:443/v3
    export OS_IDENTITY_PROVIDER=idpid
    export OS_PROTOCOL=saml
    export OS_IDENTITY_PROVIDER_URL=https://idp.example.com/idp/profile/SAML2/SOAP/ECP
    export OS_USERNAME=username
    export OS_PASSWORD=userpassword
    export OS_PROJECT_NAME=<project-name>
    export OS_PROJECT_DOMAIN_NAME=<project-domain-name>
    export OS_USER_DOMAIN_NAME=<user-domain-name>
然后执行

    (openstackclient-env)# source ~/openstack_saml2.rc
    (openstackclient-env)# openstack token issue

## 解释
