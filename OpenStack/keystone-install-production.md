
# 生产环境安装Keystone
在上一篇文章[从源码安装Keystone](./install-keystone-from-sourcecode.md)中详细介绍了Keystone的源码完整过程，但是在一个企业级应用里，不可能在生产环境中使用pip来安装依赖包，首先生产环境不一定能访问公网，另外企业无法将OpenStack与自己的部署系统集成，这对想要基于OpenStack发布产品，批量商用的企业来说是很头疼的。所以我继续进行尝试，通过将Keystone及其依赖的软件包打包成产品来部署。

## virtualenv
virtualenv通过创建独立Python开发环境的工具, 来解决依赖、版本以及间接权限问题。我希望通过virtualenv将keystone依赖的python环境独立出来，并可以移植发布。

### 安装virtualenv
    $ sudo pip install virtualenv

### 创建虚拟python环境
    $ mkdir keystone-prod
    $ virtualenv --no-site-packages venv
屏幕输出：

    New python executable in ~/keystone-prod/venv/bin/python
    Installing setuptools, pip, wheel...done.
命令virtualenv可以创建一个独立的Python运行环境，加上参数--no-site-packages，这样，已经安装到系统Python环境中的所有第三方包都不会复制过来，这样就得到了一个不带任何第三方包的“干净”的Python运行环境。

    $ cd ~/keystone-prod
    $ cp -R ../keystone ./
    $ source venv/bin/activate             
    (venv)$ cd ~/keystone
    (venv)$ pip install -r requirements.txt
执行virtualenv的activate命令后，所有命令行都带有venv前缀（即虚拟环境）。pip安装的依赖包在~/keystone-prod/venv/lib/python2.7/site-packages下。
需要格外注意的是，一定要用当前工作用户来执行pip，不能用sudo，否则安装的package仍然是系统环境python下的。

## Apache2安装和配置
为了让Apache2也可以移植，因此需要从源码编译安装Apache2。从[Apache2 Download](http://httpd.apache.org/download.cgi)源码，然后编译安装。
### 编译安装Apache2
将源码目录下载在~/keystone-prod目录下。

    $ cd ～/keystone-prod/httpd-2.4.27
    $ sudo apt-get install libapr1-dev libaprutil1-dev
    $ sudo apt-get install libpcre3-dev
    $ ./configure --prefix=～/keystone-prod/Apache2
    $ make
    $ make install
编译安装完成，--prefix指定了安装的位置，执行时请修改为绝对路径。
进入～/keystone-prod/Apache2/conf/httpd.conf，修改监听端口号为8888（本机80端口需要root用户），以及ServerName为本机IP。

    Listen 8888
    
    ServerName 192.168.1.103:8888
执行～/keystone-prod/Apache2/bin/apachectl -k start 命令，启动Apache2。在浏览器中输入 htp://192.168.1.103:8888/，显示 It works! 说明Apache2安装成功。

### 安装mod-wsgi
下载mod-wsgi源码，地址https://pypi.python.org/pypi/mod_wsgi ，pypi上有很详细的安装指南。
mod-wsgi官网也有很详细的介绍，http://modwsgi.readthedocs.io/en/develop/index.html 。
源码下载至~/keystone-prod目录。安装mod-wsgi时需要使用启动virtualenv。

    $ tar xvf mod_wsgi-4.5.19.tar.gz
    $ rm mod_wsgi-4.5.19.tar.gz
    $ cd mod_wsgi-4.5.19
    $ source ../venv/bin/activate
    (venv)$ export APXS=~/keystone-prod/Apache2/bin/apxs
    (venv)$ python setup.py install
注意：APXS环境变量导出时要修改为绝对路径。
安装完成后,mod-wsgi安装在~/keystone-prod/venv/lib/python2.7/site-packages/mod_wsgi-4.5.19-py2.7-linux-i686.egg。

验证mod-wsgi是否正常安装，可以执行下面的命令：

    (venv)$ mod_wsgi-express start-server
    Server URL         : http://localhost:8000/
    Server Root        : /tmp/mod_wsgi-localhost:8000:1000
    Server Conf        : /tmp/mod_wsgi-localhost:8000:1000/httpd.conf
    Error Log File     : /tmp/mod_wsgi-localhost:8000:1000/error_log (warn)
    Request Capacity   : 5 (1 process * 5 threads)
    Request Timeout    : 60 (seconds)
    Startup Timeout    : 15 (seconds)
    Queue Backlog      : 100 (connections)
    Queue Timeout      : 45 (seconds)
    Server Capacity    : 20 (event/worker), 20 (prefork)
    Server Backlog     : 500 (connections)
    Locale Setting     : zh_CN.UTF-8
在浏览器中输入http://localhost:8000/ 显示一条装在酒瓶里的蛇的图片，以及“My web site runs on Malt Whiskey”，说明mod-wsgi安装成功！
这种方式运行的mod-wsgi实际上是把Apache2在背后拉起，生成临时的配置文件。执行CTRL+C停止运行。

## 运行keystone
### 安装keystone
    $ cd ~/keystone-prod
    $ source ./venv/bin/activate
    $ cd ./keystone
    $ python setup.py install
执行完成后，keystone的package被安装在~/keystone-prod/venv/lib/python2.7/site-packages/目录下。
    
### 创建keystone配置文件。
    $ cd ~/keystone-prod
    $ mkdir -p ./etc/keystone
    $ cp ./keystone/etc/keystone.conf.sample ./etc/keystone/keystone.conf
    $ cp ./keystone/etc/keystone-paste.ini ./etc/keystone/
打开keystone.conf。修改database配置。搜索[database, 将默认的配置注释，替换成下面的内容：

    [database]
    connection = mysql+pymysql://keystone:KEYSTONE_DBPASS@controller/keystone 
注意KEYSTONE_DBPASS替换为创建keystone数据库实例的密码，controller替换成本机的IP地址。
修改token生成方式 搜索[token，将默认的配置注释，社区的安装指南是要求用fernet token，为了简化操作，先用uuid，后面再调整为fernet。

    [token] provider = uuid

### 安装keystone的启动环境
    $ cd ~/keystone-prod
    $ source ./venv/bin/activate
    (venv)$ mod_wsgi-express setup-server ~/keystone-prod/venv/bin/keystone-wsgi-public \
    --host 192.168.1.103 --port 5000 --user keystone --group keystone --server-root=./etc/mod_wsgi-express-5000 \
    --setenv OS_KEYSTONE_CONFIG_DIR ~/keystone-prod/etc/keystone
    Server URL         : http://192.168.1.103:5000/
    Server Root        : ~/keystone-prod/etc/mod_wsgi-express-5000
    Server Conf        : ~/keystone-prod/etc/mod_wsgi-express-5000/httpd.conf
    Error Log File     : ~/keystone-prod/etc/mod_wsgi-express-5000/error_log (warn)
    Rewrite Rules      : ~/keystone-prod/etc/mod_wsgi-express-5000/rewrite.conf
    Environ Variables  : ~/keystone-prod/etc/mod_wsgi-express-5000/envvars
    Control Script     : ~/keystone-prod/etc/mod_wsgi-express-5000/apachectl
    Request Capacity   : 5 (1 process * 5 threads)
    Request Timeout    : 60 (seconds)
    Startup Timeout    : 15 (seconds)
    Queue Backlog      : 100 (connections)
    Queue Timeout      : 45 (seconds)
    Server Capacity    : 20 (event/worker), 20 (prefork)
    Server Backlog     : 500 (connections)
    Locale Setting     : zh_CN.UTF-8

    (venv)$ ./etc/mod_wsgi-express-5000/apachectl start
在浏览器中输入http://192.168.1.103:5000/v3，报500错误，进入./etc/mod_wsgi-express-5000/error_log，发现仍然是包依赖冲突：ContextualVersionConflict: (pika 0.11.0 (/home/lishanhang/keystone-prod/venv/lib/python2.7/site-packages), Requirement.parse('pika<0.11,>=0.9'), set(['pika-pool']))。

    (venv)$ pip install pika==0.10
再次访问http://192.168.1.103:5000/v3，成功！
但是执行POST /v3/auth/tokens，仍然报500错误，查看错误日志，是pymysql module找不到。

    (venv)$ pip install pymysql
再次执行POST /v3/auth/tokens，成功返回token！

## 移植
作为可以发布的产品，必须要具有可移植的能力。前面的操作步骤中，已经将keystone及运行环境都安装在keystone-prod目录下。为了让这个目录具有可移植，我们还需要完成一些步骤。

### 移除编译源码
    $ rm -R httpd-2.4.27/
    $ rm -R mod_wsgi-4.5.19/
    $ rm -R keystone/
    
./etc/mod_wsgi-express-5000目录是mod_wsgi-express运行后生成的，很多文件中写入来绝对路径和IP以及端口号，不具备移植性，也应该删除。

    $ rm -R ./etc/mod_wsgi-express-5000/
### 安装脚本
在~/keystone-prod目录下创建init目录，用来存放初始化安装脚本。
    
    $ cd ~/keystone-prod
    $ mkdir init
    $ vi ./init/bootstrap.sh
脚本内容如下：

'''
    
'''



