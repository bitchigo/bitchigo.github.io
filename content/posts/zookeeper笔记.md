---
title: "Zookeeper笔记"
date: 2021-02-02T22:49:58+08:00
draft: true
tags: ["笔记","zookeeper"]
categories: ["笔记"]
---

# 0.目录
[TOC]

# 1. 安装

## 1.1 创建zookeeper用户

```shell
[root@localhost ~]# adduser zookeeper
[root@localhost ~]# passwd zookeeper
Changing password for user zookeeper.
New password: 
BAD PASSWORD: The password is shorter than 8 characters
Retype new password:
```

切换到zookeeper用户登陆

## 1.2 解压jdk，zookeeper

```shell
[zookeeper@localhost ~]$ tar -zxvf apache-zookeeper-3.6.2-bin.tar.gz
[zookeeper@localhost ~]$ tar -zxvf jdk-11.0.10_linux-x64_bin.tar.gz
```

## 1.3 配置环境

### 1.3.1 配置java环境

```shel
[zookeeper@localhost ~]$ nano .bash_profile
```

添加以下内容

```shell
JAVA_HOME=/home/zookeeper/jdk-11.0.10
export JAVA_HOME

PATH=$JAVA_HOME/bin:$PATH
export PATH
```

执行以下语句使配置生效

```shell
[zookeeper@localhost ~]$ . .bash_profile
```

### 1.3.2 配置zookeeper

```shell
[zookeeper@localhost ~]$ cd apache-zookeeper-3.6.2-bin
[zookeeper@localhost apache-zookeeper-3.6.2-bin]$ mkdir data
[zookeeper@localhost apache-zookeeper-3.6.2-bin]$ cd conf
[zookeeper@localhost conf]$ cp zoo_sample.cfg zoo.cfg
[zookeeper@localhost conf]$ nano zoo.cfg
```

修改dataDir

```shell
dataDir=/home/zookeeper/apache-zookeeper-3.6.2-bin/data
```

## 1.4 启动zookeeper

```shell
[zookeeper@localhost bin]$ cd /home/zookeeper/apache-zookeeper-3.6.2-bin/bin
[zookeeper@localhost bin]$ ./zkServer.sh start
```

> zkServer其他指令
>
> ./zkServer.sh stop
>
> ./zkServer.sh status

------



# 2.zookeeper简单使用

> zkCli.sh 为客户端

```shell
./zkCil.sh -server 127.0.0.1  #连接远程服务器
```



## 2.1 创建节点

```shell
create [-s] [-e] [-c] [-t ttl] path [data] [acl]   # 创建节点 s为有序节点 e为临时节点
```
> - 如果不添加 `-s`或-`e`的话为`持久化节点`
>
> - `持久化` 表示`当前会话结束`还会`保留`
>
> - `临时节点` `不能`拥有`子节点`
>
> - 当没有data 时 值为`null`
>
> - 必须`先创建上级节点`才能创建下级节点

```shell
[zk: localhost:2181(CONNECTED) 0] create /hadoop "123456"
Created /hadoop
[zk: localhost:2181(CONNECTED) 2] create -s /a "a"
Created /a0000000001
[zk: localhost:2181(CONNECTED) 3] create -e /b "b"
Created /b
[zk: localhost:2181(CONNECTED) 4] create -se /c "c"
Created /c0000000003
```

## 2.2 获取节点

``` shell
get path -s -w # s为输出子节点数据 打印当前节点状态 w为监听器
```
> 老版本无需-s参数即可输出
```shell
[zk: localhost:2181(CONNECTED) 1] get /hadoop
123456
[zk: localhost:2181(CONNECTED) 9] get /a0000000001
a
[zk: localhost:2181(CONNECTED) 7] get /b
b
[zk: localhost:2181(CONNECTED) 10] get /c0000000003
c
[zk: localhost:2181(CONNECTED) 1] get -s /e
null
cZxid = 0x13
ctime = Tue Feb 02 21:28:22 CST 2021
mZxid = 0x13
mtime = Tue Feb 02 21:28:22 CST 2021
pZxid = 0x15
cversion = 2
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 2
```

## 2.3 更新节点

```shell
set path [-v dataVersion]
```
> 老版本为 set path [dataVersion]

```shell
[zk: localhost:2181(CONNECTED) 11] set /hadoop "233"
[zk: localhost:2181(CONNECTED) 24] set /b "23345" -v 1
version No is not valid : /b
```
> 修改数据后，`数据版本号`(dataVersion)会`+1 `
>
> 数据版本`不匹配`时 数据修改不会成功

## 2.4 删除节点

```shell
delete path [-v dataVersion]
```

```shell
[zk: localhost:2181(CONNECTED) 47] delete /b
```
> 老版本为 delete path [dataVersion]
>
> 数据版本`不匹配`时 数据修改不会成功


```shell
[zk: localhost:2181(CONNECTED) 46] delete /f
Node not empty: /f
```
删除当前路径以及该路径下的子节点


```shell
rmr path   ???????
```

## 2.5 查看节点和节点状态

```shell
stat path -w-
```
```shell
[zk: localhost:2181(CONNECTED) 50] stat /f
cZxid = 0x17
ctime = Tue Feb 02 21:31:17 CST 2021
mZxid = 0x17
mtime = Tue Feb 02 21:31:17 CST 2021
pZxid = 0x18
cversion = 1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 1
```

![image-20210202214901299](/pic/zookeeper笔记/image-20210202214901299.png)

## 2.6 查看节点列表

```shell
ls path -s  # s为输出子节点同时 打印当前节点状态
```
> 老版本`不存在-s`参数 用指令 `ls2 path` 代替
```shell
[zk: localhost:2181(CONNECTED) 2] ls /e
[a, e]
```

```shell
[zk: localhost:2181(CONNECTED) 0] ls -s /e
[a, e]
cZxid = 0x13
ctime = Tue Feb 02 21:28:22 CST 2021
mZxid = 0x13
mtime = Tue Feb 02 21:28:22 CST 2021
pZxid = 0x15
cversion = 2
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 2
```

## 2.7 监听器

```shell
get path -w   添加一个监听器
```

> 老版本为 get path watch

```shell
[zk: localhost:2181(CONNECTED) 3] get /e -w
null
[zk: localhost:2181(CONNECTED) 4] set /e "233"

WATCHER::

WatchedEvent state:SyncConnected type:NodeDataChanged path:/e
```

```shell
[zk: localhost:2181(CONNECTED) 7] stat /d -w
cZxid = 0x23
ctime = Tue Feb 02 21:53:25 CST 2021
mZxid = 0x23
mtime = Tue Feb 02 21:53:25 CST 2021
pZxid = 0x23
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0
[zk: localhost:2181(CONNECTED) 8] set /d "123"

WATCHER::

WatchedEvent state:SyncConnected type:NodeDataChanged path:/d
```

```shell
[zk: localhost:2181(CONNECTED) 11] ls /d -w
[d]
[zk: localhost:2181(CONNECTED) 13] create /d/e 

WATCHER::

WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/d
Created /d/e
```

> 监听不到子节点的子节点的变化

--------------------------------------

## 3 acl权限控制

```
scheme:id:permisson
```

- 权限模式(scheme):授权的策略
| 方案   | 描述                                             |
| :----- | ------------------------------------------------ |
| world  | 只有一个用户`anyone`,代表登陆zookeeper的所有用户 |
| ip     | 对客户端使用ip地址认证                           |
| auth   | 使用已添加认证的用户认证                         |
| digest | 使用用户名 密码方式认证                          |
- 授权对象(id):授权的对象

- 权限(permisson):授权的权限

  | 权限   | ACL简写 | 描述                                 |
  | ------ | ------- | ------------------------------------ |
  | create | c       | 可以创建节点                         |
  | delete | d       | 可以删除子节点(不包括子节点的子节点) |
  | read   | r       | 可以读取节点数据和子节点列表         |
  | write  | w       | 可以设置节点数据                     |
  | admin  | a       | 可以设置节点访问控制列表权限         |

### 3.1 相关命令

#### 3.1.1 读取权限

```shell
getAcl path
```

```shell
[zk: localhost:2181(CONNECTED) 14] getAcl /d
'world,'anyone
: cdrwa
```

#### 3.1.2 设置权限

```shell
setAcl path acl
```

```shell
[zk: localhost:2181(CONNECTED) 15] setAcl /d ip:127.0.0.1:crwda
```

#### 3.1.2.1 world

```shell
setAcl path world:anyone:crwda
```

#### 3.1.2.2 ip

```shell
setAcl path ip:192.168.1.1:crwda
```

#### 3.1.2.3 auth

需要先添加授权用户

```shell
addauth digest username:password
setAcl path auth:username:crwda
```

#### 3.1.2.4 digest

密码需要先经过SHA1和BASE64处理

可以通过以下语句获得

```shell
echo -n username:password |openssl dgest -binary -sha1 |openssl base64
```

```shell
setAcl path digest:usernmae:password:crwda
```

> 需要多种权限 中间用,分割
> setAcl path world:anyone:crwda,auth:username:crwda,

---------------------------

# 4.JAVA



