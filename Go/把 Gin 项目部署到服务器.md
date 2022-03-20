# 把 Gin 项目部署到服务器

## 前言

golang使用gin开发完毕后，切不可直接运行或者使用go run xxx.go这种形式，比较正规的做法是，利用linux的服务去管理，不然ssh一退出，网站就停止了。

## 基本流程

简略流程如下：

1. build出一个可执行文件
2. 写一个sh文件，用来执行这个文件
3. 配置service
4. 启动
5. 其他配置（反向代理、ssl证书等）

## 具体做法

### 1、build 项目

我的项目中，主文件为`main.go`
执行：

```bssh
go build main.go
```

则生成了一个新文件`main`，对其设置权限，这里用了`777`，因为有时候使用`宝塔面板`的时候，它的用户是`www`.

```bash
chmod 777 main
```

### 2、写执行脚本 run.sh

新建一个文件，`vim run.sh`，里面写这样的内容：

```shell
#!/bin/bash
# 设置为 release 生产模式
export GIN_MODE=release
# 切换到路径下，这样才能够使用和开发时候一样的相对路径
cd main文件所在的绝对路径
# 启动 build 后的可执行文件
./main
```
### 3、创建一个 service 配置文件

输入命令创建：

```
vim /lib/systemd/system/mpgo.service
```

其中mpgo为服务名称，以后启动都是这个名称。

```ini
[Unit]
Description=mpgo

[Service]
Type=simple
Restart=always
RestartSec=3s
ExecStart=run.sh文件的完整路径

[Install]
WantedBy=multi-user.target
```

路径需要自行替换
说明如下：

- Description是对这个服务的描述
- Restart=always服务异常退出时会重启
- RestartSec=3s设置重启间隔为3秒
- ExecStart=run.sh文件的完整路径这个服务会执行这个文件
- WantedBy=multi-user.target所有用户都可以执行

### 4、启动命令

```bash
service mpgo start
service mpgo restart
service mpgo stop
service mpgo status
```

### 5、配置反向代理和ssl证书