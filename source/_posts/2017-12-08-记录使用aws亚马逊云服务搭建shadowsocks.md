---
title: 记录使用aws亚马逊云服务搭建shadowsocks
comments: true
toc: true
date: 2017-12-08 14:50:16
tags:
- shadowsocks
categories:
- shadowsock
description: 使用aws亚马逊云服务搭建shadowsocks教程
---
### 注册aws账号
访问[AWS](https://aws.amazon.com/cn/)，右上角注册用户。需要绑定信用卡，要有$1的授权款。目前（2017年12月6日），亚马逊云服务可以免费使用一年，每月15G流量。如果到期，要么续费，要么销毁实例。

### 创建EC2实例
1. 登录控制台，点击EC2实例
2. 启动虚拟机
3. 定制化服务器类型（我使用的是ubuntu系统）
4. 选择实例类型（这里好像只有一个符合免费套餐）
5. 之后第三步第四步都可以默认下一步
6. 到了配置安全组时，配置所有流量和端口开放。
7. 最后生成密钥并保存到本地。作为访问云服务的凭证。
8. 启动实例后的页面中，点击获得预计费用通知，勾选接收账单提醒。防止流量异常扣费。

### 连接实例
1. 在控制台中右键实例，选择连接，获取公有dns。
2. 打开xshell，工具-添加用户密钥
3. 创建新链接，主机填公有dns
4. 点击左侧用户身份验证，用户名填ubuntu，方法public key，密钥选择2步中添加的密钥。
5. 点击链接即可。  

### 实例中安装SS
```
# 获取root权限
sudo -s

# 更新apt-get
apt-get update

# 安装python包管理工具
apt-get install python-setuptools
apt-get install python-pip

# 安装shadowsocks
pip install shadowsocks
```

### 配置Shadowsocks
```
mkdir /etc/shadowsocks
vim /etc/shadowsocks/ss.json
```
vim中
```
{
"server":"0.0.0.0",
"server_port":443,
"local_address":"127.0.0.1",
"local_port":1080,
"port_password":
{
    "8100":"password1",
    "8101":"password2"
},
"_comment":
{
    "8100":"user1",
    "8101":"user2"
},
"timeout":300,
"method":"aes-256-cfb",
"fast_open":false,
"workers": 1
}
```
配置好后启动ss
```
启动：ssserver -c /etc/shadowsocks/ss.json -d start 
停止：ssserver -c /etc/shadowsocks/ss.json -d stop 
重启：ssserver -c /etc/shadowsocks/ss.json -d restart
```
设置开机启动ss
```
vi /etc/rc.local
加入一行sudo ssserver -c /etc/shadowsocks/ss.json -d start
```
windows下载[ss客户端](https://github.com/shadowsocks/shadowsocks-windows/releases)
登录控制台获取公网ip，设置服务器、端口和密码即可。