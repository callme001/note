## cometd学习笔记

cometd是comet协议的一个实现，主要应用于web即时通信。

[关于comet的介绍](http://www.ibm.com/developerworks/cn/web/wa-lo-comet/ "ibm关于comet的说明")

[cometd项目主页](https://github.com/cometd/cometd)

[cometd 3文档主页](https://docs.cometd.org/current/reference/ "cometd3文档主页")

### 开始一个cometd项目



1人发消息 另一人接受



每次进行查询一次redis返回数量



一人广播 其它一堆人收到




### 运行官方demo


### 查看最简单的一个demo


### 提取js文件

1. 解压后进入`cometd-3.0.9\cometd-javascript\jquery`
2. 执行命令`mvn install`
3. 进入到`target`目录即可看见一个新生成的war包和一个war名字一样的目录
![](http://yotuku.cn/link?url=Vyk9H50eM&tk_plan=free&tk_storage=tietuku&tk_vuid=997cb7b8-c9fb-4951-9269-637f9b27520c&tk_time=2016111119)
4. 进入之后可以看见`jquery`和`org`两个目录,就是所需要的全部js文件了~




根据key或者username创建一个session存放于redis

返回给用户消息的时候直接取出clinet session

---

用户在握手包的时候发送登陆所需要的key到服务器，取得该key之后进入redis服务器验证，
如果验证成功，则在服务器保存一个key-session记录。

在执行向用户推送消息时，首先根据用户名或者ID获取到该用户存于服务器的key,再通过key取到用户的session执行定向推送。。。

每次重新登陆时会重新生成key，注销或者重新登陆时先把redis服务器中的key-session记录删除。