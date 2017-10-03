# Keystone容器化
## Ubuntu上安装docker
我使用的操作系统是Ubuntu 16.04 LTS，参考阿里云容器服务的[博客](https://yq.aliyun.com/articles/110806):

    $ sudo apt-get update
    $ sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
    $ curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
    $ sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
    $ sudo apt-get -y update
    $ sudo apt-get -y install docker-ce
