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

```bash
tar -zxvf libzip-1.5.1.tar.gz
cd libzip-1.5.1
./configure
make && make install
```

如提示未安装cmake，则下载cmake源码安装。安装cmake时如提示未安装c编译器，则安装gcc。

```bash
yum install gcc gcc-c++
```

5、编译安装php7：

```bash
tar -zxvf php-7.3.1.tar.gz
cd php-7.3.1
./configure --prefix=/usr/local/php-7.1.12 --enable-fpm --enable-opcache --with-config-file-path=/usr/local/php-7.1.12/etc --with-mysql=mysqlnd --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --enable-static --enable-sockets --enable-wddx --enable-zip --enable-calendar --enable-bcmath --enable-soap --with-zlib --with-iconv --with-freetype-dir --with-gd --with-jpeg-dir --with-xmlrpc --enable-mbstring --with-sqlite3 --with-curl --enable-ftp --with-mcrypt --with-openssl --disable-safe-mode --with-gettext
make && make install

```

6、编译时碰到的问题：

1. php file not found.

   通常是由php-fpm配置或者nginx为php-fpm传脚本的路径有问题，检查nginx的配置文件，php.ini和php-fpm的配置文件。

2. 编译php报错configure: error: off_t undefined

   编辑 /etc/ld.so.conf 文件，追加'

   /usr/local/lib64
   /usr/local/lib
   /usr/lib
   /usr/lib64

   '，然后执行 `ldconfig -v`。

