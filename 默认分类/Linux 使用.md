# Linux 使用

## 常用操作

### 1. **iso镜像写入优盘**

```bash
sudo dd if=~/images/ubuntu.iso of=/dev/sdb
```

### 2. **使用`growisofs`续刻光盘**

```bash
# 第一次刻录
growisofs -Z /dev/dvd -R -J /some/files
# 后续追加刻录
growisofs -M /dev/dvd -R -J /more/files
```

### 3. **Ubuntu主目录下的文档文件夹改回英文**

```bash
export LANG=en_US
xdg-user-dirs-gtk-update
export LANG=zh_CN
```

### 4. **卸载Amazon**

```bash
sudo apt-get remove unity-webapps-common 
```

### 5. **vim替换**

```bash
s/old/new/g  # 当前行替换
%s/old/new/g  # 所有行替换
50,100s/old/new/g  # 50至100行进行替换
```

### 6. **du查看某个文件或目录占用磁盘空间的大小**

```bash
du -ah --max-depth=1     # a表示显示目录下所有的文件和文件夹（不含子目录），h表示以人类能看懂的方式，max-depth表示目录的深度。
```

### 7. **wget 下载网站镜像**

```bash
wget -r -p -np -k -nc http://xxx.edu.cn 
```

- -r 表示递归下载,会下载所有的链接,不过要注意的是,不要单独使用这个参数,因为如果你要下载的网站也有别的网站的链接,wget也会把别的网站的东西下载下来,所以要加上-np这个参数  
- -np 表示不下载别的站点的链接
- -k 表示将下载的网页里的链接修改为本地链接
- -p 获得所有显示网页所需的元素,比如图片什么的
- -nc 若同一路径下存在相同文件名的文件则不再下载 
- -E  或 --html-extension   将保存的URL的文件后缀名设定为“.html” 

```bash
wget -c -t 0 -O rhel6_x86_64.iso http://zs.kan115.com:8080/rhel6_x86_64.iso
```

- -c 断点续传
- -t 0 反复尝试的次数，0为不限次数
- -O rhel6_x86_64.iso 把下载的文件命名为rhel6_x86_64.iso

### 8. **查看文件夹被线程占用情况**

   ```bash
   fuser -mv /mnt/			# 要查看的目录  方法1
   lsof /mnt				# 使用lsof命令也可以 方法2
   kill -9 XXXX			# 找到后使用kill -9 杀死
   ```

### 9. 挂载Windows共享文件夹

   ```bash
   mount -t cifs //服务器ip/共享目录 /mnt目录  -o "username=aaa,password=bbb,rw,file_mode=0666,dir_mode=0777,gid=1000,uid=1000"
   ```

   uid和gid是挂载后目录拥有者的用户id和组id

### 10. 同步两个目录

```
rsync -arvuz –delete –progress 源目录  目的目录
```

### 11. ssh 免密码登录

要实现的效果是：ServerA上的userA免密码登录ServerB上的userB用户。

首先userA登录ServerA服务器，在ServerA上生成密钥对：

ssh-keygen -t rsa  (密码为空)

会在/home/userA/.ssh目录下生成密钥对  id_rsa和id_rsa.pub

然后将公钥上传到serverB服务器，以userB用户登录，执行：

ssh-copy-id userB@serverB

输入userB的密码即可。

### 12. crontab创建定时任务

```bash
# 分 时 天  月  星期  任务名称
0 */2 * * * 每两个小时执行
50 6 * * * 每天6:50执行
0 0 1,15 * * 每月1号，15号零时执行
0 3 * * 1-5 每周一至周五3点执行
```

### 13. 清空mail

    echo ''  > /var/spool/$USER

### 14. 服务器同步时间

方法1：使用ntpdate

```
ntpdate time-server
```

方法2：使用chrony（centos 8以后默认使用）

```
sudo yum install chrony
```

如果要作为时间服务器，需要防火墙中配置放行`ntp`服务。

编辑配置文件：`/etc/chrony.conf`，配置上游服务器`server time-server`，如果作为时间服务器，还需配置`allow 192.168.1.0/24`，允许客户端联通网段。

## 常见错误

### 1. 出现“No module named sql_server.pyodbc.base”错误。

```
django-pyodbc-azure
```

### 2. mail没安装。

```shell
sudo apt install mailutils
# 安装后执行mail命令会出现/var/mail/<user>没有权限。
sudo touch /var/mail/thufir
sudo chown thufir:mail /var/mail/thufir
sudo chmod o-r /var/mail/thufir
sudo chmod g+rw /var/mail/thufir
# 然后执行mail就好了。
```

### 3.  执行yum update 时报“Error: Protected multilib versions”错误。

```bash
# 核心的命令主要是：
rpm -q package-names
rpm -e package-full-version
```

### 4. Mysq报错： Host ‘XXXX’ is blocked because of many connection errors;unblock with ‘mysqladmin flush-hosts’

原因是客户端连接错误过多，连接被阻止。解决方法：

```bash
mysqladmin flush-hosts -h$HOST -u$USER -p
```

