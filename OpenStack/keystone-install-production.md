
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
    $ source venv/bin/activate             
    (venv)$ cd ~/keystone
    (venv)$ pip install -r requirements.txt
执行virtualenv的activate命令后，所有命令行都带有venv前缀（即虚拟环境）。pip安装的依赖包在~/keystone-prod/venv/lib/python2.7/site-packages下。
需要格外注意的是，一定要用当前工作用户来执行pip，不能用sudo，否则安装的package仍然是系统环境python下的。

### 安装keystone
    $ cd ~/keystone
    $ python setup.py install
通过屏幕输出可以看到keystone已经安装到~/keystone-prod/venv/lib/python2.7/site-packages下,keystone-wsgi-admin,keystone-wsgi-public,keystone-manage安装在~/keystone-prod/venv/bin目录下。

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
编译安装完成，--prefix指定了安装的位置。
进入～/keystone-prod/Apache2/conf/httpd.conf，修改监听端口号为8888（本机80端口已被占用），以及ServerName为本机IP。

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
    (venv)$ python setup.py install
    
