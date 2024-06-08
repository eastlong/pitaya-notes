> https://github.com/JuntaoLiu01/Hadoop-Hive

## 集群搭建

#### 构建镜像

1、docker login

2、使用Docker拉取`Centos`镜像

`docker pull centos:7`

3、构建具有ssh功能的`centos-ssh`镜像,其中[Dockerfile](https://github.com/JuntaoLiu01/Hadoop-Hive/blob/master/Images/centos-ssh/Dockerfile)如下：

```bash
 FROM centos:7
 MAINTAINER 'your docker username'
 RUN yum install -y openssh-server
 RUN sed -i 's/UsePAM yes/UsePAM no/g' /etc/ssh/sshd_config
 RUN yum  install -y openssh-clients
 RUN echo "root:root" | chpasswd
 RUN echo "root   ALL=(ALL)       ALL" >> /etc/sudoers
 RUN ssh-keygen -t dsa -f /etc/ssh/ssh_host_dsa_key
 RUN ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key
 RUN mkdir /var/run/sshd
 EXPOSE 22
 CMD ["/usr/sbin/sshd", "-D"]  
```



```
 cd centos-ssh
 docker build -t "centos-ssh" . 
```



```bash
  FROM centos-ssh
 ADD jdk-8u341-linux-x64.tar.gz /usr/local/
 RUN mv /usr/local/jdk1.8.0_341 /usr/local/jdk1.8
 ENV JAVA_HOME /usr/local/jdk1.8
 ENV PATH $JAVA_HOME/bin:$PATH
 ADD hadoop-2.10.1.tar.gz /usr/local
 RUN mv /usr/local/hadoop-2.10.1 /usr/local/hadoop
 ENV HADOOP_HOME /usr/local/hadoop
 ENV PATH $HADOOP_HOME/bin:$PATH 
```



 cd centos-hadoop
 docker build -t "centos-hadoop" .



