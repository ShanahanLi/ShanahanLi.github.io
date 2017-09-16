
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

研究了一下setuptools是python的一个包构建和发布工具，git地址是https://github.com/pypa/setuptools, 但是对于keystone是怎么使用setuptools的仍然不是很清楚，不过在ＣＳＤＮ上找到一篇博客[Openstack keystone setup.py和setup.cfg的解析 ](http://blog.csdn.net/joelovegreen/article/details/46373619)，感谢博友的无私分享，写得非常清晰，我立刻明白了原理。和博客唯一有出入的就是Ｏcata版本的Ｋｅｙｓｔｏｎｅ里没有MANIFEST.in，不过没有影响，找到setup.cfg就可以了。



