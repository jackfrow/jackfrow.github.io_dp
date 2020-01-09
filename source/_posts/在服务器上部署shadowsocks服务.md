---
title: 服务端部署shadowsocks
date: 2019-11-14 14:09:01
tag: 网络
categories: 工具
---

#### 前言

​        对于技术人员来说，网上查资料解决问题已经是非常重要的一个环节，谷歌的引擎搜索准确度也是世界上的顶尖水平，但是由于国内不能直接访问谷歌，所以出现了一些可以翻墙的技术和工具,shadowsocks就是其中之一。

#### 安装shadowsocks服务

​      话不多说，直接进入正题。

##### 安装pip

​      如果你的操作系统是Centos，可以使用以下命令:

   ```python
yum install python-setuptools && easy_install pip
   ```

​     如果你的操作系统是Debian或者Ubuntu,可以使用以下命令 :

```python
apt-get install python-pip
```

#####  安装shadowsocks

```python
pip install shadowsocks
```

##### 配置**shadowsocks**

在/etc/shadowsocks 文件夹下（如果没有改文件夹，则使用 *mkdir /etc/shadowsocks* 命令新建一个文件夹）新建一个配置文件config.json，并写入如下命令：

```sh
{

"server":"0.0.0.0",

"server_port":1234,

"local_address":"127.0.0.1",

"local_port":1080,
"password":"******",

"timeout":300,

"method":"aes-256-cfb",

"fast_open":false,

"workers": 1

}
```

主要参数说明

```sh
{
"server":"",     ##服务器ip地址
"server_port":1234,  ##代理端口
"local_address":"127.0.0.1",
"local_port":1080, ##本地监听端口
"password":"******",   ##连接密码
"timeout":300,
"method":"aes-256-cfb", ##加密方式
"dast_open":false
}
```

##### shadowsocks的启动与关闭

- 使用 *ssserver -c /etc/shadowsocks/config.json -d start* 命令启动
- 使用 *netstat -tunlp* 命令查看
- 使用*ssserver -c /etc/shadowsocks/config.json -d stop*命令停止

##### 连接shadowsocks

​    下载一个shadowsocks的客户端(mac版本):[shadowsocks](https://github.com/shadowsocks/ShadowsocksX-NG/releases/)

​    打开shadowsocks的客户端,配置服务器IP地址、服务器端口，密码、以及加密方式点击确认即可

![image-20191114143926410](https://tva1.sinaimg.cn/large/006y8mN6gy1g8xk945qrpj30i60k8dh6.jpg)

