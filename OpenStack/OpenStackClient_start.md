# OpenStack Client Quick Start

## 安装OpenStack Client
虽然OpenStack社区已经为很多操作系统提供来安装源，例如Ubuntu上可以直接用下面的命令安装：

    $ sudo apt-get install openstack-client
但是我建议通过pip安装，并且利用virtualenv，“绿色”安装OpenStack Client。
第一步：创建openstack client虚拟环境，并进入虚拟环境。

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

    

    

   
