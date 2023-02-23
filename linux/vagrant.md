```shell
vagrant --version
vagrant box list

vagrant box add ADDRESS
vagrant box add ubuntu/trusty64 #映像文件
vagrant box add https://atlas.hashicorp.com/ubuntu/boxes/trusty64 #url
vagrant box add CentOS7.1 file:///D:/Work/VagrantBoxes/CentOS-7.1.1503-x86_64-netboot.box #本地box

vagrant init ubuntu/trustry64 #在当前目录创建Vagrantfile的配置文件

vagrant up #进入有Vagrantfile配置文件的目录 启动时会从网上下载box到本地

vagrant ssh #进入配置文件目录 用ssh登录VM

vagrant halt #进入配置文件目录 关闭VM

vagrant destory [name|id] #销毁vm
vagrant destroy ubuntu/trusty64

1、vagrant init：初始化配置；

2、vagrant up：启动虚拟机；

3、vagrant ssh：登录虚拟机；

4、vagrant suspend：挂起虚拟机；

5、vagrant reload：重启虚拟机；

6、vagrant halt：关闭虚拟机；

7,、vagrant status：查看虚拟机状态；

8,、vagrant destroy：删除虚拟机。
```



