
# 从源码安装Keystone
在上一篇文章[如何在Ubuntu上安装Keystone](./keystone-install-ubuntu.md)中详细介绍了Keystone的安装完整过程，可以看到大部分过程都被封装在Ubuntu的apt包管理程序里，我们无法掌握具体的细节。首先不得不说，OpenStack为了推广，在使用门槛上提供了很多封装和便捷，让入门者可以快速使用，降低学习成本，做的很好。但是在一个企业级应用里，不可能用apt install等包管理程序直接在生产环境中部署OpenStack，首先生产环境不一定能访问公网，另外企业无法将OpenStack与自己的部署系统集成，这对想要基于OpenStack发布产品，批量商用的企业来说是很头疼的。　所以我尝试一下能否直接从git获取源码来部署。

## 社区的安装指南
社区也提供了从源码安装keystone的指南：　[Setting up Keystone](https://docs.openstack.org/keystone/latest/contributor/set-up-keystone.html), 但还是依赖Python的包管理程序pip来自动化构建和安装, 过程不透明。

## 获取源码
    $ git clone https://git.openstack.org/openstack/keystone.git
    $ cd keystone

## 安装Apache2
安装Apache2的同时，同时安装apache2的mod_wsgi插件。因为我在Ubuntu上安装，RedHat上apache2的名字叫httpd。

    $ sudo apt-get install apache2 libapache2-mod-wsgi

## 安装Mysql
参考上一篇文章[如何在Ubuntu上安装Keystone](./keystone-install-ubuntu.md)。非root用户可以将命令调整为 sudo apt-get install ...

## 安装keystone
### 安装pip
pip是Python的包管理程序，python3默认自带pip，通过命令可以检查是否已安排pip
    
    $ pip --version
如果操作系统安装了多个版本的Python，还需要核实pip是否关联到了你使用的Python版本。我本机Ubuntu 16.04 LTS自带了python2.7和python3.5，默认使用python2.7，但是pip是关联到Ｐython3.5，所以我重新了安装pip，安装pip可参考[pip官网](https://pip.pypa.io/en/stable/)的安装指南。

### setup.py
在keystone的目录下，有个setup.py，setup.py是python用来打包和发布程序或者库的工具。关于setup.py的介绍在百度上可以搜索到很多，这里不再赘述。Keystone的setup.py文件特别简单：

    import setuptools
    
    setuptools.setup(
        setup_requires=['pbr>=2.0.0'],
        pbr=True)
常见的第三方库的setup.py一般都name,version,description,packages,requires,install_requires等参数，但keystone的setup.py仅仅引入了pbr。keystone目录下还有一个setup.cfg，setup.cfg提供setup.py的默认参数，setup.py先解析setup.cfg文件，然后执行相关命令。setup.py和setup.cfg都是遵循python标准库中的Distutils，而setuptools工具针对Python官方的distutils做了很多针对性的功能增强，比如依赖检查，动态扩展等。pbr是setuptools的辅助工具，由OpenStack开发，pbr最重要的功能是读取keystone目录下的requirements.txt文件，生成setup函数需要的install_requires/tests_require/dependency_links。
Keystone的requirements.txt内容如下：

    # The order of packages is significant, because pip processes them in the order
    # of appearance. Changing the order has an impact on the overall integration# process, which may cause wedges in the gate later.
    # Temporarily add Babel reference to avoid problem
    # in keystone-coverage-db CI job
    Babel!=2.4.0,>=2.3.4 # BSD
    
    pbr!=2.1.0,>=2.0.0 # Apache-2.0
    WebOb>=1.7.1 # MIT
    PasteDeploy>=1.5.0 # MIT
    Paste # MIT
    Routes>=2.3.1 # MIT
    cryptography!=2.0,>=1.6 # BSD/Apache-2.0
    six>=1.9.0 # MIT
    SQLAlchemy!=1.1.5,!=1.1.6,!=1.1.7,!=1.1.8,>=1.0.10 # MIT
    sqlalchemy-migrate>=0.11.0 # Apache-2.0
    stevedore>=1.20.0 # Apache-2.0
    passlib>=1.7.0 # BSD
    
    python-keystoneclient>=3.8.0 # Apache-2.0
    keystonemiddleware>=4.12.0 # Apache-2.0
    bcrypt>=3.1.3 # Apache-2.0
    scrypt>=0.8.0 # BSD
    oslo.cache>=1.5.0 # Apache-2.0
    oslo.concurrency>=3.8.0 # Apache-2.0
    oslo.config!=4.3.0,!=4.4.0,>=4.0.0 # Apache-2.0
    oslo.context>=2.14.0 # Apache-2.0
    oslo.messaging!=5.25.0,>=5.24.2 # Apache-2.0
    oslo.db>=4.24.0 # Apache-2.0
    oslo.i18n!=3.15.2,>=2.1.0 # Apache-2.0
    oslo.log>=3.30.0 # Apache-2.0
    oslo.middleware>=3.27.0 # Apache-2.0
    oslo.policy>=1.23.0 # Apache-2.0
    oslo.serialization!=2.19.1,>=1.10.0 # Apache-2.0
    oslo.utils>=3.20.0 # Apache-2.0
    oauthlib>=0.6.0 # BSD
    pysaml2<4.0.3,>=2.4.0 # Apache-2.0
    dogpile.cache>=0.6.2 # BSD
    jsonschema!=2.5.0,<3.0.0,>=2.0.0 # MIT
    pycadf!=2.0.0,>=1.1.0 # Apache-2.0
    msgpack-python>=0.4.0 # Apache-2.0
    osprofiler>=1.4.0 # Apache-2.0
    pytz>=2013.6 # MIT

### 安装keystone
在keystone目录下，执行setup.py的install命令：

    $ sudo python setup.py install
随着大量屏幕输出，几秒后，打屏显示安装成功。

    Installed /home/lishanhang/keystone/.eggs/pbr-3.1.1-py2.7.egg
    [pbr] Generating ChangeLog
    running install
    [pbr] Writing ChangeLog
    [pbr] Generating ChangeLog
    [pbr] ChangeLog complete (0.5s)
    [pbr] Generating AUTHORS
    [pbr] AUTHORS complete (1.0s)
    running build
    running build_py
    creating build
    creating build/lib.linux-i686-2.7
    creating build/lib.linux-i686-2.7/keystone
    ... ...
    running egg_info
    creating keystone.egg-info
    writing pbr to keystone.egg-info/pbr.json
    writing requirements to keystone.egg-info/requires.txt
    writing keystone.egg-info/PKG-INFO
    writing top-level names to keystone.egg-info/top_level.txt
    writing dependency_links to keystone.egg-info/dependency_links.txt
    writing entry points to keystone.egg-info/entry_points.txt
    [pbr] Processing SOURCES.txt
    writing manifest file 'keystone.egg-info/SOURCES.txt'
    [pbr] In git context, generating filelist from git
    warning: no previously-included files matching '*.pyc' found anywhere in distribution
    writing manifest file 'keystone.egg-info/SOURCES.txt'
    ... ...
    running install_lib
    creating /usr/local/lib/python2.7/dist-packages/keystone
    ... ...
    byte-compiling /usr/local/lib/python2.7/dist-packages/keystone/revoke/routers.py to routers.pyc
    byte-compiling /usr/local/lib/python2.7/dist-packages/keystone/revoke/__init__.py to __init__.pyc
    byte-compiling /usr/local/lib/python2.7/dist-packages/keystone/revoke/backends/base.py to base.pyc
    ... ...
    running install_egg_info
    Copying keystone.egg-info to /usr/local/lib/python2.7/dist-packages/keystone-12.0.0.0rc2.dev62-py2.7.egg-info
    running install_scripts
    Installing keystone-wsgi-admin script to /usr/local/bin
    Installing keystone-wsgi-public script to /usr/local/bin
    Installing keystone-manage script to /usr/local/bin
安装完成后，keystone的package已经加入到python发布包目录/usr/local/lib/python2.7/dist-packages，可被其他python库引用。keystone-wsgi-admin, keystone-wsgi-public, keystone-manage三个脚本安装到/usr/local/bin目录下。
但是在/usr/local/lib/python2.7/dist-packages中并没有看到keystone的第三方库安装进来，执行keystone-manage会报依赖找不到的异常。

### 安装Keystone依赖包
通过在pip install的命令安装依赖包，安装的依赖包放在Python的dist-packages目录下，注意：sudo apt-get install 安装的package存放在 /usr/lib/python2.7/dist-packages目录中；pip 或者 easy_install安装的package存放在/usr/local/lib/python2.7/dist-packages目录中；手动从源代码安装的package存放在site-packages目录中。
前面也提到了，keystone的依赖包定义在requirements.txt文件中，先进入到keystone源码目录，通过下面的命令安装：

    $ sudo pip install -r requirements.txt
安装过程中也不是一帆风顺，编译scrypt时报编译错误,可以通过下面的命令解决：

"src/scrypt.c:27:20: fatal error: Python.h: No such file or directory"，

    $ sudo apt-get install python-dev
scrypt-1.2.0/libcperciva/crypto/crypto_aes.c:6:25: fatal error: openssl/aes.h: No such file or directory

    $ sudo apt-get install libssl-dev
安装完成后，再执行keystone-manage就不会包依赖错误了。

## 配置Keystone
### 创建keystone的数据库
参考上一篇文章[如何在Ubuntu上安装Keystone](./keystone-install-ubuntu.md)。

### 创建配置目录
进入keystone源码的etc目录，执行下面的命令：

    $ sudo mkdir -p /etc/keystone
    $ sudo cp keystone.conf.sample /etc/keystone/keystone.conf
    $ sudo cp keystone-paste.ini /etc/keystone/keystone-paste.ini
    $ sudo cp logging.conf.sample /etc/keystone/logging.conf
    
### 配置keystone.conf
在/etc/keystone目录下打开keystone.conf。
1. 修改database配置。搜索\[database，将默认的配置注释，替换成下面的内容：

    [database]  
    connection = mysql+pymysql://keystone:KEYSTONE_DBPASS@controller/keystone
注意KEYSTONE_DBPASS替换为创建keystone数据库实例的密码，controller替换成本机的IP地址。
2. 修改token生成方式
搜索\[token，将默认的配置注释，社区的安装指南是要求用fernet token，但是我需要用到pki token，所以设置为pki。

   [token]
   provider = pki
   
## 配置Apache2
部分内容可参考博客：http://www.cnblogs.com/Security-Darren/p/4458728.html
配置Apache在启动的时候加载Keystone,将Keystone源码下的httpd/wsgi-keystone.conf复制到/etc/apache2/conf-available/，然后在/etc/apache2/conf-enabled/目录中创建一个指向/etc/apache2/conf-available/wsgi-keystone.conf的同名软链接:

    $ sudo cp ./httpd/wsgi-keystone.conf /etc/apache2/conf-available/
    $ cd /etc/apache2/conf-enabled/
    $ sudo ln -s /etc/apache2/conf-available/wsgi-keystone.conf wsgi-keystone.conf

目前版本的wsgi-keystone.conf文件，除了启动服务的用户从"keystone"修改为真实的用户外，其他不用修改可直接使用。修改后重启apache2，查看进程，可看到keystone已经运行!

    $ service apache2 restart
    $ ps -ef|grep keystone
    keystone+ 22592 22589  0 13:21 ?        00:00:00 (wsgi:keystone-pu -k start
    keystone+ 22593 22589  0 13:21 ?        00:00:00 (wsgi:keystone-pu -k start
    keystone+ 22594 22589  0 13:21 ?        00:00:00 (wsgi:keystone-pu -k start
    keystone+ 22595 22589  0 13:21 ?        00:00:00 (wsgi:keystone-pu -k start
    keystone+ 22596 22589  0 13:21 ?        00:00:00 (wsgi:keystone-pu -k start
    keystone+ 22597 22589  0 13:21 ?        00:00:00 (wsgi:keystone-ad -k start
    keystone+ 22598 22589  0 13:21 ?        00:00:00 (wsgi:keystone-ad -k start
    keystone+ 22599 22589  0 13:21 ?        00:00:00 (wsgi:keystone-ad -k start
    keystone+ 22600 22589  0 13:21 ?        00:00:00 (wsgi:keystone-ad -k start
    keystone+ 22601 22589  0 13:21 ?        00:00:00 (wsgi:keystone-ad -k start

## 装载数据
### keystone建表
执行keystone-manage命令：

    $ keystone-manage db_sync
命令执行成功后，进入Mysql，可以看到所有表都已经创建成功。表都是空的，还没有插入初始化数据。

    $ sudo mysql
    MariaDB [(none)]> use keystone;
    MariaDB [keystone]> show tables;
    
### 初始化数据
执行keystone-manage的bootstrap命令，完成初始化的用户导入：

    # keystone-manage bootstrap --bootstrap-password ADMIN_PASS \
    --bootstrap-admin-url http://controller:35357/v3/ \
    --bootstrap-internal-url http://controller:5000/v3/ \
    --bootstrap-public-url http://controller:5000/v3/ \
    --bootstrap-region-id RegionOne
注意，命令中的controller需要替换成本机IP地址，　ADMIN_PASS需要替换成自定义的密码。 bootstrap过程中往keystone的表中插入了一些数据，例如Default domain, admin project。

## 验证
在浏览器上输入http://controller:5000/v3 错误，打开/var/log/apache2/keystone.log发现错误信息：

    ContextualVersionConflict: (pika 0.11.0 (/usr/local/lib/python2.7/dist-packages), Requirement.parse('pika<0.11,>=0.9'), set(['pika-pool']))
进入到/usr/local/lib/python2.7/dist-packages目录，果然pika版本号是0.11，通过pip命令安装0.10版本。

    $ sudo pip install pika==0.10
安装完成后，原来的0.11版本会被0.10版本覆盖。重启Apache2，再次访问http://controller:5000/v3 ，已经能正常访问。

    {"version": {"status": "stable", "updated": "2017-02-22T00:00:00Z", "media-types": [{"base": "application/json", "type": "application/vnd.openstack.identity-v3+json"}], "id": "v3.8", "links": [{"href": "http://localhost:5000/v3/", "rel": "self"}]}}



