

## RockerMq安装

### 前期准备

1、安装jdk

参考：[centos环境下java安装](https://blog.csdn.net/axing2015/article/details/83614800)



### 安装RockerMq

1、下载

https://rocketmq.apache.org/dowloading/releases/



2、解压

```sh
tar -zxvf rocket-mq-.....
```

3、重命名

```
mv rocketmq-all-5.2.0-bin-release rocketmq-all-5.2.0
```

4、配置环境变量

```
vim /etc/profile

# 加入如下值
export NAMESRV_ADDR=阿里云公网IP:9876
```

5、修改broker的配置文件

```bash
namesrvAddr=localhost:9876
autoCreateTopicEnable=true
brokerIP1=阿里云公网IP
```

6、启动

Tips: 请先创建好log文件夹

```
nohup sh bin/mqnamesrv > ./logs/namesrv.log &

nohup sh bin/mqbroker -c conf/broker.conf > ./logs/broker.log &
```

查看启动结果：

```
jps