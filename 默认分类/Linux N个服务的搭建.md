# inux N个服务的搭建

搭建开发测试环境非常麻烦，公司的开发测试服务器中毒做数据恢复，顺带整理下搭建的方式。

**本人是程序员，linux系统知识比较薄弱，故系统的安全方面本文未涉及，请酌情做安全策略**。

本来是内网服务器，用frp暴露到了公网，导致被挖矿。。感觉frp还是不安全，现在只能在需要穿透的 时候开下，其他时候关闭。

> mysql数据备份和恢复参考：[juejin.im/post/5d8b85…](https://juejin.im/post/5d8b8587f265da5b752598a1)

> [SpringBoot整合jwt和mybatis-plus的脚手架项目](https://juejin.im/post/5d80e107f265da039c63a6d8)

## 基本配置

### 安装基本命令

有些命令可能未默认安装，如果发现命令无法使用，再通过下面的方式进行安装。

#### 安装ifconfig

centos 7中自带的查看网络的命令是: ip addr

如果还是想要 ifconfig

安装net-tools

```
yum install net-tools
```

#### 安装vim

```
yum install vim
```

### 网络配置

> 如果是虚拟机模式，VM box的网络模式修改为`桥接`。

#### 修改hostname

#### 修改ip地址

命令为：

```
[root@localhost ~]# vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
复制代码
```

修改为如下即可，然后重启网卡`service network restart`：

```
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static"                   ###修改为static模式才能配置ip，默认是dhcp模式
IPADDR="192.168.1.254"               ###网卡IP地址
BROADCAST-"192.168.1.255"            ###子网广播地址
GATEWAY="192.168.1.1"				 ###网关地址
NETMASK="255.255.255.0"				 ###网卡对应网络掩码
DNS1="192.168.1.1"					 ###DNS地址
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="enp0s3"
UUID="226a0768-3a2f-4485-9694-d8fea85694ad"
DEVICE="enp0s3"
ONBOOT="yes"            #系统启动时是否设置此网络接口，设置为yes时，系统启动时激活此设备。默认设置为yes。
复制代码
```

#### 修改dns

##### 1） vi /etc/resolv.conf

```
[root@localhost ~]# vi /etc/resolv.conf
# Generated by NetworkManager
nameserver 192.168.1.1        #本机的网关地址（路由器的地址），在ip配置的时候有指定
nameserver 114.114.114.114    #其他dns
naemserver 1.1.1.1

search localdomain
复制代码
```

##### 3）确保可用DNS解析

[root@localhost Desktop]# grep hosts /etc/nsswitch.conf

### 配置更新源

配置更新源为阿里源

新建`sourceSet.sh`文件，贴上如下代码执行即可：

> chmod 775 sourceSet.sh

```
#!/bin/bash
#########################################
#Function:    update source
#Usage:       bash update_source.sh
#Author:      Customer service department
#Company:     Alibaba Cloud Computing
#Version:     5.0
#########################################

check_os_release()
{
  while true
  do
    os_release=$(grep "Red Hat Enterprise Linux Server release" /etc/issue 2>/dev/null)
    os_release_2=$(grep "Red Hat Enterprise Linux Server release" /etc/redhat-release 2>/dev/null)
    if [ "$os_release" ] && [ "$os_release_2" ]
    then
      if echo "$os_release"|grep "release 5" >/dev/null 2>&1
      then
        os_release=redhat5
        echo "$os_release"
      elif echo "$os_release"|grep "release 6" >/dev/null 2>&1
      then
        os_release=redhat6
        echo "$os_release"
      else
        os_release=""
        echo "$os_release"
      fi
      break
    fi
    os_release=$(grep "Aliyun Linux release" /etc/issue 2>/dev/null)
    os_release_2=$(grep "Aliyun Linux release" /etc/aliyun-release 2>/dev/null)
    if [ "$os_release" ] && [ "$os_release_2" ]
    then
      if echo "$os_release"|grep "release 5" >/dev/null 2>&1
      then
        os_release=aliyun5
        echo "$os_release"
      elif echo "$os_release"|grep "release 6" >/dev/null 2>&1
      then
        os_release=aliyun6
        echo "$os_release"
      elif echo "$os_release"|grep "release 7" >/dev/null 2>&1
      then
        os_release=aliyun7
        echo "$os_release"
      else
        os_release=""
        echo "$os_release"
      fi
      break
    fi
    os_release_2=$(grep "CentOS" /etc/*release 2>/dev/null)
    if [ "$os_release_2" ]
    then
      if echo "$os_release_2"|grep "release 5" >/dev/null 2>&1
      then
        os_release=centos5
        echo "$os_release"
      elif echo "$os_release_2"|grep "release 6" >/dev/null 2>&1
      then
        os_release=centos6
        echo "$os_release"
      elif echo "$os_release_2"|grep "release 7" >/dev/null 2>&1
      then
        os_release=centos7
        echo "$os_release"
      else
        os_release=""
        echo "$os_release"
      fi
      break
    fi
    os_release=$(grep -i "ubuntu" /etc/issue 2>/dev/null)
    os_release_2=$(grep -i "ubuntu" /etc/lsb-release 2>/dev/null)
    if [ "$os_release" ] && [ "$os_release_2" ]
    then
      if echo "$os_release"|grep "Ubuntu 10" >/dev/null 2>&1
      then
        os_release=ubuntu10
        echo "$os_release"
      elif echo "$os_release"|grep "Ubuntu 12.04" >/dev/null 2>&1
      then
        os_release=ubuntu1204
        echo "$os_release"
      elif echo "$os_release"|grep "Ubuntu 12.10" >/dev/null 2>&1
      then
        os_release=ubuntu1210
        echo "$os_release"
     elif echo "$os_release"|grep "Ubuntu 14.04" >/dev/null 2>&1
     then
        os_release=ubuntu1204
        echo "$os_release" 
      else
        os_release=""
        echo "$os_release"
      fi
      break
    fi
    os_release=$(grep -i "debian" /etc/issue 2>/dev/null)
    os_release_2=$(grep -i "debian" /proc/version 2>/dev/null)
    if [ "$os_release" ] && [ "$os_release_2" ]
    then
      if echo "$os_release"|grep "Linux 6" >/dev/null 2>&1
      then
        os_release=debian6
        echo "$os_release"
      elif echo "$os_release"|grep "Linux 7" >/dev/null 2>&1
      then
        os_release=debian7
        echo "$os_release"
      else
        os_release=""
        echo "$os_release"
      fi
      break
    fi
    os_release=$(grep -i "opensuse" /etc/issue 2>/dev/null)
    os_release_2=$(grep -i "opensuse" /etc/*release 2>/dev/null)
    if [ "$os_release" ] && [ "$os_release_2" ]
    then
      if echo "$os_release"|grep "openSUSE 13.1" >/dev/null 2>&1
      then
        os_release=opensuse1301
        echo "$os_release"
      else
        os_release=""
        echo "$os_release"
      fi
      break
    fi
    break
    done
}

modify_aliyun5_yum()
{
  wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-5.repo
  sed -i 's/\$releasever/5/' /etc/yum.repos.d/CentOS-Base.repo
  wget -qO /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-5.repo
  yum clean metadata
  yum makecache
  cd ~
}

modify_rhel5_yum()
{
  wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-5.repo
  wget -qO /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-5.repo
  yum clean metadata
  yum makecache
  cd ~
}

modify_rhel6_yum()
{
  wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
  wget -qO /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo
  yum clean metadata
  yum makecache
  cd ~
}

modify_rhel7_yum()
{
  wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
  wget -qO /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
  yum clean metadata
  yum makecache
  cd ~
}

update_ubuntu10_apt_source()
{
echo -e "\033[40;32mBackup the original configuration file,new name and path is /etc/apt/sources.list.back.\n\033[40;37m"
cp -fp /etc/apt/sources.list /etc/apt/sources.list.back
cat > /etc/apt/sources.list <<EOF
#ubuntu
deb http://cn.archive.ubuntu.com/ubuntu/ maverick main restricted universe multiverse
deb-src http://cn.archive.ubuntu.com/ubuntu/ maverick main restricted universe multiverse
#163
deb http://mirrors.163.com/ubuntu/ maverick main universe restricted multiverse
deb-src http://mirrors.163.com/ubuntu/ maverick main universe restricted multiverse
deb http://mirrors.163.com/ubuntu/ maverick-updates universe main multiverse restricted
deb-src http://mirrors.163.com/ubuntu/ maverick-updates universe main multiverse restricted
#lupaworld
deb http://mirror.lupaworld.com/ubuntu/ maverick main universe restricted multiverse
deb-src http://mirror.lupaworld.com/ubuntu/ maverick main universe restricted multiverse
deb http://mirror.lupaworld.com/ubuntu/ maverick-security universe main multiverse restricted
deb-src http://mirror.lupaworld.com/ubuntu/ maverick-security universe main multiverse restricted
deb http://mirror.lupaworld.com/ubuntu/ maverick-updates universe main multiverse restricted
deb http://mirror.lupaworld.com/ubuntu/ maverick-proposed universe main multiverse restricted
deb-src http://mirror.lupaworld.com/ubuntu/ maverick-proposed universe main multiverse restricted
deb http://mirror.lupaworld.com/ubuntu/ maverick-backports universe main multiverse restricted
deb-src http://mirror.lupaworld.com/ubuntu/ maverick-backports universe main multiverse restricted
deb-src http://mirror.lupaworld.com/ubuntu/ maverick-updates universe main multiverse restricted
EOF
apt-get update
}

update_ubuntu1204_apt_source()
{
echo -e "\033[40;32mBackup the original configuration file,new name and path is /etc/apt/sources.list.back.\n\033[40;37m"
cp -fp /etc/apt/sources.list /etc/apt/sources.list.back
cat > /etc/apt/sources.list <<EOF
#12.04
deb http://mirrors.aliyun.com/ubuntu/ precise main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ precise-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ precise-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ precise-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ precise-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ precise main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ precise-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ precise-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ precise-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ precise-backports main restricted universe multiverse
EOF
apt-get update
}

update_ubuntu1210_apt_source()
{
echo -e "\033[40;32mBackup the original configuration file,new name and path is /etc/apt/sources.list.back.\n\033[40;37m"
cp -fp /etc/apt/sources.list /etc/apt/sources.list.back
cat > /etc/apt/sources.list <<EOF
#12.10
deb http://mirrors.aliyun.com/ubuntu/ quantal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ quantal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ quantal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ quantal-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ quantal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ quantal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ quantal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ quantal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ quantal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ quantal-backports main restricted universe multiverse
EOF
apt-get update
}

update_ubuntu1404_apt_source()
{
echo -e "\033[40;32mBackup the original configuration file,new name and path is /etc/apt/sources.list.back.\n\033[40;37m"
cp -fp /etc/apt/sources.list /etc/apt/sources.list.back
cat > /etc/apt/sources.list <<EOF
#14.04
deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
EOF
apt-get update
}

update_debian6_apt_source()
{
echo -e "\033[40;32mBackup the original configuration file,new name and path is /etc/apt/sources.list.back.\n\033[40;37m"
cp -fp /etc/apt/sources.list /etc/apt/sources.list.back
cat > /etc/apt/sources.list <<EOF
#debian6
deb http://mirrors.aliyun.com/debian/ squeeze main non-free contrib
deb http://mirrors.aliyun.com/debian/ squeeze-proposed-updates main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ squeeze main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ squeeze-proposed-updates main non-free contrib
EOF
apt-get update
}

update_debian7_apt_source()
{
echo -e "\033[40;32mBackup the original configuration file,new name and path is /etc/apt/sources.list.back.\n\033[40;37m"
cp -fp /etc/apt/sources.list /etc/apt/sources.list.back
cat > /etc/apt/sources.list <<EOF
#debian7
deb http://mirrors.aliyun.com/debian/ wheezy main non-free contrib
deb http://mirrors.aliyun.com/debian/ wheezy-proposed-updates main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ wheezy main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ wheezy-proposed-updates main non-free contrib
EOF
apt-get update
}

update_opensuse_source()
{
  mv /etc/zypp/repos.d/* /tmp/
  zypper addrepo -f http://mirrors.aliyun.com/opensuse/distribution/13.1/repo/oss/ openSUSE-13.1-Oss
  zypper addrepo -f http://mirrors.aliyun.com/opensuse/distribution/13.1/repo/non-oss/ openSUSE-13.1-Non-Oss
  zypper addrepo -f http://mirrors.aliyun.com/opensuse/update/13.1/ openSUSE-13.1-Update-Oss
  zypper addrepo -f http://mirrors.aliyun.com/opensuse/update/13.1-non-oss/ openSUSE-13.1-Update-Non-Oss
  zypper addrepo -f http://mirrors.aliyun.com/opensuse/distribution/13.1/repo/oss/ openSUSE-13.1-Oss-aliyun
  zypper addrepo -f http://mirrors.aliyun.com/opensuse/distribution/13.1/repo/non-oss/ openSUSE-13.1-Non-Oss-aliyun  zypper addrepo -f http://mirrors.aliyun.com/opensuse/update/13.1/ openSUSE-13.1-Update-Oss-aliyun
  zypper addrepo -f http://mirrors.aliyun.com/opensuse/update/13.1-non-oss/ openSUSE-13.1-Update-Non-Oss-aliyun
}

####################Start###################
#check lock file ,one time only let the script run one time 
LOCKfile=/tmp/.$(basename $0)
if [ -f "$LOCKfile" ]
then
  echo -e "\033[1;40;31mThe script is already exist,please next time to run this script.\n\033[0m"
  exit
else
  echo -e "\033[40;32mStep 1.No lock file,begin to create lock file and continue.\n\033[40;37m"
  touch $LOCKfile
fi

#check user
if [ $(id -u) != "0" ]
then
  echo -e "\033[1;40;31mError: You must be root to run this script, please use root to install this script.\n\033[0m"
  rm -rf $LOCKfile
  exit 1
fi
echo -e "\033[40;32mStep 2.Begin to check the OS issue.\n\033[40;37m"
os_release=$(check_os_release)
if [ "X$os_release" == "X" ]
then
  echo -e "\033[1;40;31mThe OS does not identify,So this script is not executede.\n\033[0m"
  rm -rf $LOCKfile
  exit 0
else
  echo -e "\033[40;32mThis OS is $os_release.\n\033[40;37m"
fi

echo -e "\033[40;32mStep 3.Begin to modify the source configration file and update.\n\033[40;37m"
case "$os_release" in
aliyun5)
  modify_aliyun5_yum
  ;;
redhat5|centos5)
  modify_rhel5_yum
  ;;
redhat6|centos6|aliyun6)
  modify_rhel6_yum
  ;;
centos7|aliyun7)
  modify_rhel7_yum
  ;;
ubuntu10)
  update_ubuntu10_apt_source
  ;;
ubuntu1204)
  update_ubuntu1204_apt_source
  ;;
ubuntu1210)
  update_ubuntu1210_apt_source
  ;;
ubuntu1404)
  update_ubuntu1404_apt_source
  ;;
debian6)
  update_debian6_apt_source
  ;;
debian7)
  update_debian7_apt_source
  ;;
opensuse1301)
  update_opensuse_source
  ;;
esac
echo -e "\033[40;32mSuccess,exit now!\n\033[40;37m"
rm -rf $LOCKfile

复制代码
```

## 开发环境

### JDK

##### 下载jdk

最新jdk1.8.0_211我已经上传到网盘：

链接：[pan.baidu.com/s/1B9DRL5iZ…](https://link.juejin.im/?target=https%3A%2F%2Fpan.baidu.com%2Fs%2F1B9DRL5iZsPzqn1hVN325kw) 提取码：5e92 复制这段内容后打开百度网盘手机App，操作更方便哦)

上传和解压到该路径： `/usr/local/base/jdk1.8.0_211`

##### 设置环境变量：

vi /etc/profile 在最后添加如下内容：

```
#java environment
export JAVA_HOME=/data/jdk1.8.0_211
export CLASSPATH=.:${JAVA_HOME}/jre/lib/rt.jar:${JAVA_HOME}/lib/dt.jar:${JAVA_HOME}/lib/tools.jar
export PATH=$PATH:${JAVA_HOME}/bin
复制代码
```

##### 重新加载环境变量：

```
[root@izwz9hy3mj62nle7573jv5z jdk1.8.0_181]# source /etc/profile
[root@izwz9hy3mj62nle7573jv5z jdk1.8.0_181]# java -version
java version "1.8.0_181"
Java(TM) SE Runtime Environment (build 1.8.0_181-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.181-b13, mixed mode)
复制代码
```

> **卸载jdk：**
>
> 如果需要卸载，那么删除环境变量和jdk解压后的目录即可。

## 安装服务

### nginx

#### 添加源

```
# rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
复制代码
```

#### 安装

```
yum install -y nginx
复制代码
```

#### 查看安装后的目录

```
# whereis nginx
nginx: /usr/sbin/nginx /usr/lib64/nginx /etc/nginx /usr/share/nginx /usr/share/man/man8/nginx.8.gz /usr/share/man/man3/nginx.3pm.gz
复制代码
```

- Nginx配置路径：**/etc/nginx/**
- 执行程序路径：**/usr/sbin/nginx**
- PID目录：/var/run/nginx.pid
- 错误日志：/var/log/nginx/error.log
- 访问日志：/var/log/nginx/access.log
- 默认站点目录：/usr/share/nginx/html

需要主要的是`配置路径`和`执行程序路径`。

#### 启停命令

```
#启动
[root@nginx]#/usr/sbin/nginx -c /etc/nginx/nginx.conf
#检测配置
[root@nginx]#/usr/sbin/nginx -c /etc/nginx/nginx.conf -t
#重启
[root@nginx]# /usr/sbin/nginx -s reload
[root@nginx]# /usr/sbin/nginx -c /etc/nginx/nginx.conf -s reload

复制代码
```

#### 测试

如果显示表示成功

```
[root@zhirui-base nginx]# curl localhost:80
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
    <head>
        <title>Test Page for the Nginx HTTP Server on Fedora</title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
        <style type="text/css">
       ...
复制代码
```

当然也可以在浏览器打开网址测试。

记得防火墙放开端口！，CentOS7防火墙操作参考：[juejin.im/post/5d3b26…](https://juejin.im/post/5d3b264551882504b7122127)

nginx配置文件配置参考：

[juejin.im/post/5d7e3f…](https://juejin.im/post/5d7e3f51f265da03a31d687b)

[juejin.im/post/5d8190…](https://juejin.im/post/5d81906c518825300a3ec7ca)

### mysql

#### 安装：

这里安装的是mariadb，mariadb和mysql是可以通用的，是mysql的开源分支，比mysql更加有前景。

```
# yum install mariadb-server mariadb 
复制代码
```

#### 配置配置文件：

```
#vim /etc/my.cnf
[mysqld]
character-set-server = utf8    #设置默认编码, 在[mysqld]下配置,[client][mysql]不配置!!!
lower_case_table_names = 1   #配置大小写不敏感, 查询时不区分大小写, 1:不区分, 0:区分
group_concat_max_len = 204800  #修改最大返回字符串的长度
复制代码
```

#### 启停操作：

```
systemctl start mariadb  #启动MariaDB
systemctl stop mariadb  #停止MariaDB
systemctl restart mariadb  #重启MariaDB
systemctl enable mariadb  #设置开机启动
复制代码
```

#### 配置帐号和权限

第一次登陆的时候不需要密码

```
# mysql -uroot -p
use mysql;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;
update user set password=password("123456") where user='root';
flush privileges;
exit
复制代码
```

#### 添加端口到防火墙，并重启防火墙：

```
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload
复制代码
```

> **如果需要卸载使用如下方式：**
>
> 参考：[www.cnblogs.com/javahr/p/92…](https://link.juejin.im/?target=https%3A%2F%2Fwww.cnblogs.com%2Fjavahr%2Fp%2F9245443.html)
>
> 1. 使用以下命令查看当前安装mysql情况，查找以前是否装有mysql
>
> ```
> `rpm -qa|grep -i mysql`
> 复制代码
> ```
>
>   可以看到如下图的所示：
>
> 
>
> ![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="563" height="83"></svg>)
>
> 
>
>   显示之前安装了：
>
>     `MySQL-client-5.5.25a-1.rhel5`
>
>    `MySQL-server-5.5.25a-1.rhel5`
>
> 1. 停止mysql服务、删除之前安装的mysql
>
>   删除命令：**rpm -e –nodeps 包名**
>
> ```
> &emsp;&emsp;rpm -ev --nodeps MySQL-client-5.5.25a-1.rhel5
> &emsp;&emsp;rpm -ev --nodeps MySQL-server-5.5.25a-1.rhel5
> 复制代码
> ```
>
> 1. 查找之前老版本mysql的目录、并且删除老版本mysql的文件和库**
>
> ```
> `find / -name mysql`
> 复制代码
> ```
>
>   查找结果如下：
>
> ```
> `find / -name mysql` `/var/lib/mysql``/var/lib/mysql/mysql``/usr/lib64/mysql  `
> 复制代码
> ```
>
>   删除对应的mysql目录
>
> ```
> `rm -rf /var/lib/mysql``rm -rf /var/lib/mysql``rm -rf /usr/lib64/mysql`
> 复制代码
> ```
>
>   具体的步骤如图：查找目录并删除
>
> 
>
> ![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="566" height="134"></svg>)
>
> 
>
>   **注意：**卸载后/etc/my.cnf不会删除，需要进行手工删除
>
> ```
> `rm -rf /etc/my.cnf`
> 复制代码
> ```
>
> 1. 再次查找机器是否安装mysql
>
> ```
> `rpm -qa|grep -i mysql`
> 复制代码
> ```
>
> 如果是yum命令安装的还需要执行如下命令：
>
> yum remove mariadb*
>
> yum remove mysql*

### redis

#### 安装一个仓库

为了能够实现yum命令安装，故先需要安装该仓库

```
yum install epel-release
复制代码
```

#### 安装redis数据库

```
`yum ``install` `redis`
复制代码
```

#### 安装完毕后，使用下面的命令启动redis服务

```
`# 启动redis``service redis start``# 停止redis``service redis stop``# 查看redis运行状态``service redis status``# 查看redis进程``ps` `-ef | ``grep` `redis`
复制代码
```

#### 设置redis为开机自动启动

```
`chkconfig redis on`
复制代码
```

#### 进入redis服务

```
`# 进入本机redis``redis-cli``# 列出所有key``keys *`
复制代码
```

#### 修改配置

##### 打开配置文件

```
`vi` `/etc/redis``.conf`
复制代码
```

##### 修改默认端口

查找 port 6379 修改为相应端口即可

##### 修改默认密码

查找 requirepass foobared 将 foobared 修改为你的密码

##### 允许远程访问

```
# 找到 bind 127.0.0.1 将其注释
# 找到 protected-mode yes 将其改为
protected-mode no
复制代码
```

### nexus

#### 安装

官网地址：[www.sonatype.com/download-os…](https://link.juejin.im/?target=https%3A%2F%2Fwww.sonatype.com%2Fdownload-oss-sonatype)

```
# cd /opt
# wget https://download.sonatype.com/nexus/3/nexus-3.2.0-01-unix.tar.gz
# tar zxvf nexus-3.2.0-01-unix.tar.gz
```

解压后，在当前目录中除了nexus-3.2.0-01还有一个`sonatyoe-work`目录，用户存放仓库数据的，可根据需要将其改为其他路径，或使用软链接的方式。 这里说下通过改配置文件的方式，将其改为其他路径吧。 查看nexus-3.2.0-01/bin/nexus.vmoptions文件:

```
# vim /opt/nexus-3.2.0-01/bin/nexus.vmoptions
```

分别对应着以下属性，有需求可以修改：

```
-XX:LogFile=../sonatype-work/nexus3/log/jvm.log
-Dkaraf.data=../sonatype-work/nexus3
-Djava.io.tmpdir=../sonatype-work/nexus3/tmp
```

`sonatype-work/nexus3/etc`的目录下有个配置文件nexus.properties，可以配置对应的ip地址和端口

用vim打开文件：

```
vim nexus.properties
```

默认是如下配置，如果ip冲突可以按需修改端口等：

```
# Jetty section
# application-port=8081
# application-host=0.0.0.0
# nexus-args=${jetty.etc}/jetty.xml,${jetty.etc}/jetty-http.xml,${jetty.etc}/jetty-requestlog.xml
# nexus-context-path=/
...
```

> 也可以在nexus-3.2.0-01/bin/nexus.rc上指定新的帐号运行nexus。
>
> 编辑nexus.rc：
>
> ```
> run_as_user="nexus"
> ```
>
> 那么linux系统中需要添加一个叫做`nexus`的用户，用来启动nexus。

#### 配置

访问nexus：[http://serverip:8081](https://link.juejin.im/?target=http%3A%2F%2Fserverip%3A8081%2F)，配置之前需要先登录

默认帐号是`admin`，默认密码是`admin123`。

##### 配置`maven-central`

修改central仓库的远程仓库地址(建议修改成spring或者阿里云的仓库)

仓库地址如下：

```
1.http://repo1.maven.org/maven2 （官方，速度一般）
2.http://maven.aliyun.com/nexus/content/repositories/central/ (阿里云,速度快)
3.http://repository.jboss.com/maven2/
4.<https://repository.sonatype.org/content/groups/public/> 
5.http://mvnrepository.com/
```

修改后rebuild下index

##### 配置第三方仓库

该仓库用于上传私有jar用。

点击`Create repository`填写名称`3rd-repo`，其他默认即可。

##### 配置`maven-public`

`maven-public`是nexus的中心组，我们使用的nexus的url填写的就是这个组的地址。所以我们需要在这里将刚刚创建的仓库添加到这个组里面。

选中，添加到右边即可：

##### maven部署到nexus

我在这里整理了几种部署到maven的方式：[juejin.im/post/5d8b7a…](https://juejin.im/post/5d8b7a40f265da5ba273ac05)

### git

安装jinkens之前需要安装git，直接用yum命令进行安装

```
# yum install git
```

安装后需要通过如下方式找到git程序位置，后面的jenkins需要使用。

```
# find / -name git
/usr/bin/git
```

可以查看到git所在位置为`/usr/bin/git`。

查看git的版本

```
# git --version
git version 1.8.3.1
```

### maven

#### 安装

安装jinkens之前需要安装maven

下载页：[maven.apache.org/download.cg…](https://link.juejin.im/?target=http%3A%2F%2Fmaven.apache.org%2Fdownload.cgi)

下载和安装：

```
wget http://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.2/binaries/apache-maven-3.6.2-bin.tar.gz
tar -zxf apache-maven-3.6.2-bin.tar.gz
mv apache-maven-3.6.2
cd apache-maven-3.6.2/conf
vim settings.xml 
```

#### settings.xml配置

在conf目录下，有个settings.xml，在使用前需要进行配置。

> 如果配置后jenkins无法构建，请参考：《[maven配置：jenkins的生产环境](https://juejin.im/post/5d8ee32af265da5b5d203d3d)》

需要配置的几个配置项：

1. 配置下载jar的存储路径

```
<localRepository>/path/to/local/repo</localRepository>
```

1. `<mirrors>` `</mirrors>`下配置仓库地址

   > 我在jenkins构建的时候，同时配置了下面两个仓库，程序需要的私服私有jar一直跑去阿里云下载，然后提示下载不下来。如果出现这种情况，请只保留私服的仓库地址试试。

```
<!-- 私服地址 -->
<mirror>
	<id>com.zhirui.group</id>
	<mirrorOf>central</mirrorOf>
	<name>com.zhirui.group</name>
	<url>http://192.168.1.254:8081/repository/maven-public/</url>
</mirror>
<!-- 阿里云的仓库地址 -->
<mirror>
	<id>alimaven</id>
	<name>aliyun maven</name>
	<url>http://maven.aliyun.com/nexus/content/groups/public/</url>
	<mirrorOf>central</mirrorOf>
</mirror>
```

### jenkins

jenkins安装比较复杂，我另外写了篇文章来详细讲解如果安装和配置，点击查看《[jenkins自动部署Spring Cloud服务实战](https://juejin.im/post/5d8b2466f265da5b633cc37d)》

### jira

jira安装比较复杂，我另外写了篇文章来详细讲解如果安装和配置，点击查看《[jenkins自动部署Spring Cloud服务实战](https://juejin.im/post/5d8cd84be51d4578045a3522)》

### gitblit

几大代码管理工具对比，这里只讲gitblit：

![img](Linux N个服务的搭建.assets/16d7623fdd9e7191)

#### 官网地址：

[www.gitblit.com/](https://link.juejin.im/?target=http%3A%2F%2Fwww.gitblit.com%2F)

#### 安装：

```
wget http://dl.bintray.com/gitblit/releases/gitblit-1.8.0.tar.gz
```

#### 修改端口：

```
server.httpPort = 7000 
server.httpsPort = 7443
```

#### 开始访问

url：[http://192.168.1.234:7000/](https://link.juejin.im/?target=http%3A%2F%2F192.168.1.234%3A7000%2F)

默认账号密码：admin/admin

### frp

frp用于内网穿透用，可以实现在公网访问内网的服务。

#### 下载

```
wget https://github.com/fatedier/frp/releases/download/v0.29.0/frp_0.29.0_linux_amd64.tar.gz
```

> 更多的版本下载：[github.com/fatedier/fr…](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Ffatedier%2Ffrp%2Freleases)

#### 服务器端

##### 配置

```
# 配置和frp客户端连接用
[common]
bind_port = 7000
token = javasea@frp

dashboard_port = 7557
#仪表板的用户名和密码都是可选的，如果没有设置，默认是admin。
dashboard_user = admin
dashboard_pwd = javasea@frpdash
```

##### 启动

```
# frps -c frps.ini
```

#### 客户端

##### 配置

```
# 配置和frp服务端连接用
[common]
server_addr = 120.79.246.166
server_port = 7000
token = javasea@frp   #用于和服务器端认证
# mysql暴露到公网
[mysql]
type = tcp
local_port = 3306
remote_port = 7575
# gitblit暴露到公网
[gitblit]
type = tcp
local_port = 7000
remote_port = 7576
```

##### 启动

```
# frpc -c frpc.ini
```

##### 管理页面：

url: [http://192.168.1.254:7557](https://link.juejin.im/?target=http%3A%2F%2F192.168.1.254%3A7557)， 7557就是上面服务器端配置的dash端口。