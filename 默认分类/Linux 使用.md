# Linux 使用

## 常用操作

1. **iso镜像写入优盘**

   sudo dd if=~/images/ubuntu.iso of=/dev/sdb

2. **Ubuntu主目录下的文档文件夹改回英文**

```
export LANG=en_US
xdg-user-dirs-gtk-update
export LANG=zh_CN
```

3. 卸载Amazon

```
sudo apt-get remove unity-webapps-common 
```

4. vim替换

```
s/old/new/g  # 当前行替换
%s/old/new/g  # 所有行替换
50,100s/old/new/g  # 50至100行进行替换
```

5. du查看某个文件或目录占用磁盘空间的大小

```
du -ah --max-depth=1     # a表示显示目录下所有的文件和文件夹（不含子目录），h表示以人类能看懂的方式，max-depth表示目录的深度。
```

6. wget 下载网站镜像

```
wget -r -p -np -k -nc http://xxx.edu.cn 
```

- -r 表示递归下载,会下载所有的链接,不过要注意的是,不要单独使用这个参数,因为如果你要下载的网站也有别的网站的链接,wget也会把别的网站的东西下载下来,所以要加上-np这个参数  
- -np 表示不下载别的站点的链接
- -k 表示将下载的网页里的链接修改为本地链接
- -p 获得所有显示网页所需的元素,比如图片什么的
- -nc 若同一路径下存在相同文件名的文件则不再下载 
- -E  或 --html-extension   将保存的URL的文件后缀名设定为“.html” 

```
wget -c -t 0 -O rhel6_x86_64.iso http://zs.kan115.com:8080/rhel6_x86_64.iso
```

- -c 断点续传

- -t 0 反复尝试的次数，0为不限次数
- -O rhel6_x86_64.iso 把下载的文件命名为rhel6_x86_64.iso

7. 

## 常见错误

1. 出现“No module named sql_server.pyodbc.base”错误。

```
django-pyodbc-azure
```

2. mail没安装。

```shell
sudo apt install mailutils
# 安装后执行mail命令会出现/var/mail/<user>没有权限。
sudo touch /var/mail/thufir
sudo chown thufir:mail /var/mail/thufir
sudo chmod o-r /var/mail/thufir
sudo chmod g+rw /var/mail/thufir
# 然后执行mail就好了。
```

3. 执行yum update 时报“Error: Protected multilib versions”错误。

   ```bash
   # 核心的命令主要是：
   rpm -q package-names
   rpm -e package-full-version
   ```

   