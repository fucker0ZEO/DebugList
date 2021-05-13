# Nacos在Ubuntu下启动失败 |Debug日志

## 现象

在nacos/bin目录下使用`sh shartup.sh -m standalone`命令启动报错：

`startup.sh: 130: startup.sh: [[: not found`

![image-20210513192735117](Nacos%E5%9C%A8Ubuntu%E4%B8%8B%E5%90%AF%E5%8A%A8%E5%A4%B1%E8%B4%A5%20Debug%E6%97%A5%E5%BF%97.assets/image-20210513192735117.png)

看似成功，但是 `ps -ef |grep nacos`查不到启动的nacos进程。于是使用`tail -f /opt/tmp/nacos/logs/start.out`查看nacos的log

```bash
Error：Could not create the Java Virtual Machine.
Error:A Fatal exception has occurred,Program will exit.
```

报错无法创建JVM

## 解决问题

这个错误很熟悉，一般原因都是JDK配置的环境变量的锅。

于是查看 `/etc/profile`这个文件

![image-20210513193725590](Nacos%E5%9C%A8Ubuntu%E4%B8%8B%E5%90%AF%E5%8A%A8%E5%A4%B1%E8%B4%A5%20Debug%E6%97%A5%E5%BF%97.assets/image-20210513193725590.png)

仔细检查环境变量后，确定环境变量没有问题。于是初步预测可能和WSL2有关，于是用 `WSL2 Nacos`作为关键词检索，相关信息少的可怜...并无这个错误。于是又以 `Nacos startup.sh: 130: startup.sh: [[: not found` 这个错误信息为关键词检索，找到了答案：

![image-20210513194157135](Nacos%E5%9C%A8Ubuntu%E4%B8%8B%E5%90%AF%E5%8A%A8%E5%A4%B1%E8%B4%A5%20Debug%E6%97%A5%E5%BF%97.assets/image-20210513194157135.png)



[Nacos部署中的一些常见问题汇总-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/702000)

想不到居然和ubuntu的脚本启动方式有关！需要改变sh为bash -f启动脚本。命令如下：

`bash -f ./startup.sh -m standalone`

然后就一切正常了

![image-20210513194606546](Nacos%E5%9C%A8Ubuntu%E4%B8%8B%E5%90%AF%E5%8A%A8%E5%A4%B1%E8%B4%A5%20Debug%E6%97%A5%E5%BF%97.assets/image-20210513194606546.png)

## 最后

localhost在windows中是可用的。Windows可以通过localhost访问到wsl2中启动的服务！！很类似本地访问

![image-20210513205931396](Nacos%E5%9C%A8Ubuntu%E4%B8%8B%E5%90%AF%E5%8A%A8%E5%A4%B1%E8%B4%A5%20Debug%E6%97%A5%E5%BF%97.assets/image-20210513205931396.png)

此外，为了验证wsl2端是否出现问题。安装了命令行浏览器。w3m和lynx

```bash
#安装w3m
sudo apt install w3m w3m-img
#访问网站的方式，以baidu为例
w3m baidu.com
#安装lynx
sudo apt install lynx
#访问网站的方式
lynx baidu.com
#退出的方式都是q
```

或许具体的图片无法展示，但是文字还是可以看的，可以验证本机能否访问

![image-20210513210555842](Nacos%E5%9C%A8Ubuntu%E4%B8%8B%E5%90%AF%E5%8A%A8%E5%A4%B1%E8%B4%A5%20Debug%E6%97%A5%E5%BF%97.assets/image-20210513210555842.png)

