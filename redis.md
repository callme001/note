## 安装 

#### ubuntu apt安装

```
sudo apt-get install redis-server
```

配置文件路径在```/etc/redis/```下

#### 源码编译安装

 - 解压安装包
 - ```tar -xvf redis.tar.gz ```
 - 进入解压目录
 - ```sudo make```
 - ```sudo make install ``` 

默认安装到```/usr/local/bin```
编译安装需要将redis配置文件拷贝到```/etc/redis/```下


#### 启动服务

查看端口是否被占用

``` netstat –ntlp |grep 6379 ```

干掉进程```sudo kill -9 pid```

启动

 - 默认配置启动 ```redis-server & ```后台启动
 - 指定配置文件启动 ```redis-server /etc/redis/redis.conf```

客户端检测连接
``` redis-cli ```

出现```127.0.0.1:6379>```启动成功

停止服务
```redis-cli shutdown```
如果不是默认端口 可指定端口 ```redis-cli 6380 shutdown```

设置后台运行redis
```sudo vim /etc/redis/redis.conf```

将```daemonize```设置成``` yes```



#### 外部连接

使用指定配置文件启动