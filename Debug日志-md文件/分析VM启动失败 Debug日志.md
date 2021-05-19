---
typora-copy-images-to: 分析VM启动失败 Debug日志.assets
---

# 分析VM启动失败 |Debug日志



```
Error occurred during initialization of VM
Could not reserve enough space for object heap
```

内存实际使用情况

![image-20210519195748812](%E5%88%86%E6%9E%90VM%E5%90%AF%E5%8A%A8%E5%A4%B1%E8%B4%A5%20Debug%E6%97%A5%E5%BF%97.assets/image-20210519195748812.png)



浏览器居然出现了闪退，本来以为是配置出错。但是结合实际内存使用来看确实可能是物理机内存不足！



2个IDEA关闭后

![image-20210519200112947](%E5%88%86%E6%9E%90VM%E5%90%AF%E5%8A%A8%E5%A4%B1%E8%B4%A5%20Debug%E6%97%A5%E5%BF%97.assets/image-20210519200112947.png)



再次启动

![image-20210519200457838](%E5%88%86%E6%9E%90VM%E5%90%AF%E5%8A%A8%E5%A4%B1%E8%B4%A5%20Debug%E6%97%A5%E5%BF%97.assets/image-20210519200457838.png)

Unable to allocate 195648KB bitmaps for parallel garbage collection for the requested 6260736KB heap.



于此同时，再次出现黑屏+l浏览器闪退

再次打开IDEA+打开浏览器，同时AIDA64监控内存占用。物理内存经过内存整理后，降低到65%占用，但此时虚拟内存的占用已经达到100%，还剩几十兆，这时我点了打开打开项目。下一秒屏幕整体黑屏，操作系统执行内存回收....

回收后

![image-20210519203212872](%E5%88%86%E6%9E%90VM%E5%90%AF%E5%8A%A8%E5%A4%B1%E8%B4%A5%20Debug%E6%97%A5%E5%BF%97.assets/image-20210519203212872.png)



重启系统后再运行，启动成功

![image-20210519211247748](%E5%88%86%E6%9E%90VM%E5%90%AF%E5%8A%A8%E5%A4%B1%E8%B4%A5%20Debug%E6%97%A5%E5%BF%97.assets/image-20210519211247748-1621429984281.png)