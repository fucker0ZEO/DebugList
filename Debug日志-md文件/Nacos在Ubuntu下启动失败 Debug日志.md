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