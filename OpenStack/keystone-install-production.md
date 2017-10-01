
# 生产环境安装Keystone
在上一篇文章[从源码安装Keystone](./install-keystone-from-sourcecode.md)中详细介绍了Keystone的源码完整过程，但是在一个企业级应用里，不可能在生产环境中使用pip来安装依赖包，首先生产环境不一定能访问公网，另外企业无法将OpenStack与自己的部署系统集成，这对想要基于OpenStack发布产品，批量商用的企业来说是很头疼的。所以我继续进行尝试，通过将Keystone及其依赖的软件包打包成产品来部署。

## virtualenv
virtualenv通过创建独立Python开发环境的工具, 来解决依赖、版本以及间接权限问题。我希望通过virtualenv将keystone依赖的python环境独立出来，并打包发布。

### 安装virtualenv
    $ sudo pip install virtualenv

### 创建生产目录
    $ mkdir keystone-prod
    $ virtualenv --no-site-packages venv
    New python executable in /home/lishanhang/keystone-prod/venv/bin/python
    Installing setuptools, pip, wheel...done.
命令virtualenv就可以创建一个独立的Python运行环境，加上参数--no-site-packages，这样，已经安装到系统Python环境中的所有第三方包都不会复制过来，这样，我们就得到了一个不带任何第三方包的“干净”的Python运行环境。

    lishanhang@lishanhang-K42JE:~/keystone-prod$ source venv/bin/activate
    (venv) lishanhang@lishanhang-K42JE:~/keystone-prod$ 
    $ cp -r ~/keystone ./
    $ cd ./keystone
    $ pip install -r requirements.txt

## 安装依赖包

### 可选2: pip安装依赖包到指定目录
通过在pip install的命令中指定安装参数，将依赖包安装到指定的目录下：

    $ pip install --target=/home/community_work/keystone-packages -r requirements.txt

pip install执行成功后，在对应的目录下就能看到所有安装的依赖包。
在碰到问题搜索解决答案时，有博友提到可通过virtualenv隔离环境依赖的办法，有空的时候要研究下，http://bbs.chinaunix.net/forum.php?mod=viewthread&tid=4142313
