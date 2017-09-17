
# 从源码安装Keystone
在上一篇文章[如何在Ubuntu上安装Keystone](./keystone-install-ubuntu.md)中详细介绍了Keystone的安装完整过程，可以看到大部分过程都被封装在Ubuntu的apt包管理程序里，我们无法掌握具体的细节。首先不得不说，OpenStack为了推广，在使用门槛上提供了很多封装和便捷，让入门者可以快速使用，降低学习成本，做的很好。但是在一个企业级应用里，不可能用apt install等包管理程序直接在生产环境中部署OpenStack，首先生产环境不一定能访问公网，另外企业无法将OpenStack与自己的部署系统集成，这对想要基于OpenStack发布产品，批量商用的企业来说是很头疼的。　所以我尝试一下能否直接从git获取源码来部署。

## 社区的安装指南
社区也提供了从源码安装keystone的指南：　[Setting up Keystone](https://docs.openstack.org/keystone/latest/contributor/set-up-keystone.html), 但还是依赖Python的包管理程序pip来自动化构建和安装, pip同样需要访问公网，　过程也不透明。

## 获取源码
    $ git clone https://git.openstack.org/openstack/keystone.git
    $ cd keystone

## 找到包依赖关系
在keystone目录下找到setup.py,内容特别简单，引用setuptools，　调用setup方法，结束...

    # THIS FILE IS MANAGED BY THE GLOBAL REQUIREMENTS REPO - DO NOT EDIT
    import setuptools

    # In python < 2.7.4, a lazy loading of package `pbr` will break
    # setuptools if some other modules registered functions in `atexit`.
    # solution from: http://bugs.python.org/issue15881#msg170215
    try:
        import multiprocessing  # noqa
    except ImportError:
        pass

    setuptools.setup(
        setup_requires=['pbr>=2.0.0'],
        pbr=True)

研究了一下setuptools是python的一个包构建和发布工具，git地址是https://github.com/pypa/setuptools, 但是对于keystone是怎么使用setuptools的仍然不是很清楚，不过在ＣＳＤＮ上找到一篇博客[Openstack keystone setup.py和setup.cfg的解析 ](http://blog.csdn.net/joelovegreen/article/details/46373619)，感谢博友的无私分享，写得非常清晰，我立刻明白了原理。和博客唯一有出入的就是Ｏcata版本的Ｋｅｙｓｔｏｎｅ里没有MANIFEST.in，不过没有影响，找到setup.cfg就可以
从博客里可以看到，keystone依赖的package列在requirements.txt文件中：

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

## 安装依赖包
### 关于pip
pip是Ｐｙｔｈｏｎ的包管理程序，python3默认自带pip，通过命令可以检查是否已安排pip
    
    $ pip --version
如果操作系统安装了多个版本的Python，还需要核实pip是否关联到了你使用的Python版本。我本机Ubuntu 16.04 LTS自带了python2.7和python3.5，默认使用python2.7，但是ｐｉｐ是关联到Ｐython3.5，所以我重新了安装pip，安装pip可参考[pip官网](https://pip.pypa.io/en/stable/)的安装指南。

### 可选１：IDE自动安装依赖包
我本机的python IDE是PyCharm，非常强大的ＩＤＥ，打开keystone代码目录后，Pycharm能识别自动requirements.txt，并提示是否导入依赖，选择是后，Pycharm列出依赖包，确认后开始自动安装。唯一麻烦的是，Pycharm是以工作用户运行的，安装依赖需要用ｒｏｏｔ权限执行python，所以经常需要sudo密码。Pycharm使用的python版本是默认的（我本机Python2.7），安装的依赖包放在Python的dist-packages目录下，注意：

    sudo apt-get install 安装的package存放在 /usr/lib/python2.7/dist-packages目录中
    pip 或者 easy_install安装的package存放在/usr/local/lib/python2.7/dist-packages目录中
    手动从源代码安装的package存放在site-packages目录中

我本机的/usr/local/lib/python2.7/dist-packages在安装依赖前只有pip和wheel，现在已经有非常多的package。
这种方法的缺点是依赖了ＩＤＥ，如果是在开发环境，这种方法很方便，很省心。 如果开发环境和生产环境是同一个系统的话，可以把dist-packages目录下的package拷贝过去。

### 可选2: pip安装依赖包到指定目录
通过在pip install的命令中指定安装参数，将依赖包安装到指定的目录下：

    $ pip install --target=/home/community_work/keystone-packages -r requirements.txt

安装过程中也不是一帆风顺，编译scrypt时报编译错误,可以通过下面的命令解决：

"src/scrypt.c:27:20: fatal error: Python.h: No such file or directory"，

    $ sudo apt-get install python-dev
    $ sudo apt-get install python3-dev

scrypt-1.2.0/libcperciva/crypto/crypto_aes.c:6:25: fatal error: openssl/aes.h: No such file or directory

    $ sudo apt-get install libssl-dev

pip install执行成功后，在对应的目录下就能看到所有安装的依赖包。
在碰到问题搜索解决答案时，有博友提到可通过virtualenv隔离环境依赖的办法，有空的时候要研究下，http://bbs.chinaunix.net/forum.php?mod=viewthread&tid=4142313
