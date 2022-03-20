# Centos 7 离线安装PHP7

1、下载PHP安装包

2、安装依赖包

```bash
yum install -y libxml2-devel libtool* curl-devel libjpeg-devel libpng-devel freetype-devel openssl openssl-devel libzip-devel
```

3、编译安装 libmcrypt 包：

```bash
tar -zxvf libmcrypt-2.5.8.tar.gz 
cd libmcrypt-2.5.8
./configure && make && make install
```

4、编译安装 libzip 包：



4、编译安装php7：

cd php-7.3.1

