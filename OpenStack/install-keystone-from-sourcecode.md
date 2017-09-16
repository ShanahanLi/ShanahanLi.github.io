
# 从源码安装Keystone
在上一篇文章[如何在Ubuntu上安装Keystone](./keystone-install-ubuntu.md)中详细介绍了Keystone的安装完整过程，可以看到大部分过程都被封装在Ubuntu的apt包管理程序里，我们无法掌握具体的细节。首先不得不说，OpenStack为了推广，在使用门槛上提供了很多封装和便捷，让入门者可以快速使用，降低学习成本，做的很好。但是在一个企业级应用里，不可能用apt install等包管理程序直接在生产环境中部署OpenStack，首先生产环境不一定能访问公网，另外企业无法将OpenStack与自己的部署系统集成，这对想要基于OpenStack发布产品，批量商用的企业来说是很头疼的。　所以我尝试一下能否直接从git获取源码来部署。

## 社区的安装指南
社区也提供了从源码安装keystone的指南：　[Setting up Keystone](https://docs.openstack.org/keystone/latest/contributor/set-up-keystone.html)

## 找到包依赖关系

感谢博友的无私分享[Setup.py](http://blog.csdn.net/joelovegreen/article/details/46373619)


