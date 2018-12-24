# [如何将django部署从顶级目录迁移到子目录下(NGINX UWSGI DJANGO)](https://www.cnblogs.com/aguncn/p/6437847.html)

因为公司网站合并，要将我们的DJANGO项目从IP的顶级目录迁移到域名的二级目录。

以前硬编码的URL可惨了。

还涉及到upload目录，静态目录，websocket目录.

全用{% url %}问题不太大。

nginx分前后两级，uwsgi配置要作相应更改，django的setting需要变量登陆网址。

这样，在正式网站访问二级目录，测试环境仍然可以根目录访问。

nginx_front:

```
server {
        listen       80;
        server_name  localhost;
        
        location /prism/ {
            proxy_redirect    off;
            proxy_set_header Host $host;
   	    proxy_set_header X-Real-IP $remote_addr;
	    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://prism_host;
	    client_max_body_size          1000m;
	    client_body_timeout           5m;
	    proxy_connect_timeout         5m;
	    proxy_read_timeout            5m;
	    proxy_send_timeout            5m;
        }
	location /prism/websocket {
            proxy_redirect    off;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $host;
            proxy_pass http://websocket_host;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }
	
    }
```

nginx_back:

```
server {
        listen       80;
        server_name  localhost;
       
        location /prism/ {            
            include  uwsgi_params;
            uwsgi_pass  prism_host;
	    uwsgi_param SCRIPT_NAME /prism; 
	    uwsgi_modifier1 30;
            index  index.html index.htm;
	    client_max_body_size          1000m;
            client_body_timeout           5m;
            proxy_connect_timeout         5m;
            proxy_read_timeout            5m;
            proxy_send_timeout            5m;
        }
	location /prism/ws_log {
	    proxy_redirect    off;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $host;
            proxy_pass http://websocket_host;
	}

	location ^~ /prism/static {
            	alias /Prism/static;
        }
    }
```

uwsgi.ini

```
[uwsgi]
socket = 10.1.1.11:9090
chdir = /Prism
module = settings.wsgi
master = true
vhost = true
no-stie = true
workers = 4
reload-mercy = 10
vacuum = true    
max-requests = 1000
limit-as = 512
buffer-sizi = 30000
pidfile = /var/log/prism/uwsgi9090.pid   
daemonize = /var/log/prism/uwsgi9090.log
listen=1024
```

setting.py(生产)--测试的settings.py不用变更

```
LOGIN_URL = '/prism/accounts/login'
STATIC_URL = '/prism/static/'
```

然后，大功告成！

![img](https://images2015.cnblogs.com/blog/465438/201702/465438-20170224113746023-1132000191.png)

 