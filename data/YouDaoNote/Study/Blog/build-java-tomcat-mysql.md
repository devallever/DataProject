---
title: Ubuntu server 下配置JDK，Tomcat，MySQL
date: 2017-05-16 15:56:38
tags:
 - Java
 - Tomcat
 - MySQL
 - Ubuntu
categories: [Ubuntu]
---

# 配置JDK环境
> 参考：[http://blog.csdn.net/yebhweb/article/details/55098189](http://blog.csdn.net/yebhweb/article/details/55098189)

## 安装
下载好jdk包，如：jdk-8u121-linux-x64.tar.gz
为了同一管理安装目录统一为：/opt/
复制压缩包到 /opt/jdk目录下
进入到 /opt/jdk
解压：tar -xvf jdk-8u121-linux-x64.tar.gz
删除压缩包

## 配置jdk环境
修改文件进行全局配置
```
gedit/.bashrc
```
在最后加入如下内容
打开之后在末尾添加
```
export JAVA_HOME=/usr/lib/jdk/jdk1.8.0_121  
export JRE_HOME=${JAVA_HOME}/jre  
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib  
export PATH=${JAVA_HOME}/bin:$PATH
```

请记住，在上述添加过程中，等号两侧不要加入空格，不然会出现“不是有效的标识符”，因为source /etc/profile 时不能识别多余到空格，会理解为是路径一部分。
然后保存。

使配置文件生效
```
source ~/.bashrc
```


检验是否安装成功
在终端输入如下命令
```
java -version
```

# 安装Tomcat
下载压缩包，如：apache-tomcat-8.5.11.tar.gz
复制到：/opt/tomcat
进入到目录：/opt/tomcat
解压：
tar -xvf apache-tomcat-8.5.11.tar.gz
删除压缩包
进入到tomcat安装目录下的bin目录
执行：startup.sh就可以启动Tomcat

#安装MySQL
命令安装
```
apt install mysql-server-5.7
```
按提示即可安装完成，并且自动启动MySQL
使用root登录mysql
```
mysql -u root -p
```
退出：ctrl+z

## 解决中文乱码
修改配置文件
```
vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

在[mysqld]中加入如下内容
```
character-set-server = utf8
```
重启MySQL：systomctl restart mysql.service
启动MySQL：systemctl start mysql.service
查看mysql编码
```
show variables like 'character%';
```
显示已经改为utf8
## 创建数据库设置字符集
```
create database dbname default charset utf8 COLLATE utf8_general_ci;
```

导入sql文件
```
source path/filename.sql;
```

## 解决MySQL查表大小写问题
修改配置文件：
/etc/my.cnf 或  
/etc/mysql/mysql.conf.d/mysqld.cnf  
在[mysqld]下添加
```
lower_case_table_names=1
```
1：表示忽略大小写
0：表示不忽略大小写



