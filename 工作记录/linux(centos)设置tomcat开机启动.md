# [linux(centos)设置tomcat开机启动](https://www.cnblogs.com/taomylife/p/7992533.html)

## **方法一，/etc/rc.d/rc.local：**


linux 下tomcat开机自启动
修改Tomcat/bin/startup.sh 为:

```
export JAVA_HOME=/usr/java/j2sdk1.4.2_08
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:.
export PATH=$PATH:$JAVA_HOME/bin
export CATALINA_HOME=/usr/local/tomcat
/usr/local/tomcat/bin/catalina.sh start
```

在/etc/rc.d/rc.local中加入:

```
/usr/local/tomcat/bin/startup.sh
```

## 方法二，添加为系统服务：

**以nginx为例**

vim /usr/lib/systemd/system/nginx.service

```
[Unit]
Description=nginx - high performance web server
Documentation=http://nginx.org/en/docs/
After=network-online.target remote-fs.target nss-lookup.target
 
[Service]
Type=forking
PIDFile=/usr/local/nginx/logs/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t 
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s stop
PrivateTmp=true
 
[Install]
WantedBy=multi-user.target
```

检查是否成功

```
systemctl daemon-reload
#启动：
systemctl start nginx
# 重启
systemctl restart nginx
#停止
systemctl stop nginx
#设置开启启动
systemctl enable nginx
# 查看服务是否启动，端口是否打开
netstat -tulnp
```

**自定义一个web服务：xxx**

```
vim /usr/lib/systemd/system/xxx.service
 
[Unit]
Description=ywbuild
After=network.target remote-fs.target nss-lookup.target
 
[Service]
Type=forking
ExecStart=/usr/local/bin/uwsgi --ini /opt/server/uwsgi.ini  # 自定义自己服务启动命令
User=root
Group=root
 
[Install]
WantedBy=multi-user.target
 
#启动：
systemctl start xxx
#设置开启启动
systemctl enable xxx
```

