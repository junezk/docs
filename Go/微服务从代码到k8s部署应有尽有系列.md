# [微服务从代码到k8s部署应有尽有系列](https://segmentfault.com/a/1190000041685798)

我们用一个系列来讲解从需求到上线、从代码到k8s部署、从日志到监控等各个方面的微服务完整实践。

整个项目使用了go-zero开发的微服务，基本包含了go-zero以及相关go-zero作者开发的一些中间件，所用到的技术栈基本是go-zero项目组的自研组件，基本是go-zero全家桶了。

实战项目地址：[https://github.com/Mikaelemmmm/go-zero-looklook](https://github.com/Mikaelemmmm/go-zero-looklook)

## 一、项目简介

整个项目使用了go-zero开发的微服务，基本包含了go-zero以及相关go-zero作者开发的一些中间件，所用到的技术栈基本是go-zero项目组的自研组件，基本是go-zero全家桶了。

项目目录结构如下：

- app：所有业务代码包含api、rpc以及mq（消息队列、延迟队列、定时任务）
- common：通用组件 error、middleware、interceptor、tool、ctxdata等
- data：该项目包含该目录依赖所有中间件(mysql、es、redis、grafana等)产生的数据，此目录下的所有内容应该在git忽略文件中，不需要提交。
- deploy：
  - filebeat: docker部署filebeat配置
  - go-stash：go-stash配置
  - nginx: nginx网关配置
  - prometheus ： prometheus配置
  - script：
    - gencode：生成api、rpc，以及创建kafka语句，复制粘贴使用
    - mysql：生成model的sh工具
  - goctl: 该项目goctl的template，goctl生成自定义代码模版，tempalte用法可参考go-zero文档，复制到家目录下.goctl即可，
    该项目用到goctl版本是v1.3.0
- doc : 该项目系列文档

## 二、用到技术栈

- go-zero
- nginx网关
- filebeat
- kafka
- go-stash
- elasticsearch
- kibana
- prometheus
- grafana
- jaeger
- go-queue
- asynq
- asynqmon
- dtm
- docker
- docker-compose
- mysql
- redis

## 三、项目架构图

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041685800.png)

## 四、业务架构图

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041685801.jpeg)

## 五、项目环境搭建

本项目采用air热加载功即时修改代码及时生效，并且不需要每次都要重启，改了代码自动就在容器中重新加载了，本地不需要启动服务，本地安装的sdk就是写代码自动提示使用的，实际运行是以来容器中cosmtrek/air的golang环境。所以使用goland、vscode都一样

### 1、clone代码&更新依赖

```shell
$ git clone git@github.com:Mikaelemmmm/go-zero-looklook.git
$ go mod tidy
```

### 2、启动项目所依赖的环境

```shell
$ docker-compose -f docker-compose-env.yml up -d
```

jaeger: [http://127.0.0.1:16686/search](https://link.segmentfault.com/?enc=0vOpO9uQ09ZKQ%2BeiCsaRRw%3D%3D.3Frozv%2BYqumX99t4Brom9cDJbaxOBUDhHARXV5NLp%2Fo%3D)

asynq （延时、定时消息队列）: [http://127.0.0.1:8980/](https://link.segmentfault.com/?enc=4cBtQe3KHzKBNtdr6irwnA%3D%3D.2c6rQ99R2eZ4Lr9G%2BnyyFvXCdY4746e8QbTdRHBa%2Fr0%3D)

kibana: [http://127.0.0.1:5601/](https://link.segmentfault.com/?enc=JuwKnZeKdu74QelijxgvdA%3D%3D.%2BLNYbeb7rjgfsEuEZeiUjoXUN84TTdBCj7cw9dvAsLk%3D)

Elastic search: [http://127.0.0.1:9200/](https://link.segmentfault.com/?enc=nQkZECC4p2poXOzK6as%2F%2Fg%3D%3D.fQWI0YHFsZ9WvWHkvY8hsMHzbLgJQwCdnuci9RmhiP4%3D)

Prometheus: [http://127.0.0.1:9090/](https://link.segmentfault.com/?enc=GZSJc4TQ4bG%2FTPapLb8T0A%3D%3D.ZXak5EbIr7mJouGopGlFe9zSdyJ2mvB5fs0oMjP8rzM%3D)

Grafana: [http://127.0.0.1:3001/](https://link.segmentfault.com/?enc=6wD9WRmxP13RUN9jNdCu8w%3D%3D.ZknUYz8jEcHUFztFLBoMsW8evYNGY9SXnJ%2FqR9Ee5Mk%3D)

Mysql: 自行客户端工具(Navicat、Sequel Pro)查看

- host : 127.0.0.1
- port : 33069
- username : root
- pwd : PXDN93VRKUm8TeE7

Redis: 自行工具（redisManager）查看

- host : 127.0.0.1
- port : 63799
- pwd : G62m50oigInC30sf

Kafka: 自行客户端工具查看

- host : 127.0.0.1
- port : 9092

### 3、拉取项目依赖镜像

因为本项目是用air热加载的，所以是在air+golang镜像中运行，直接docker-compose也可以，但是考虑依赖可能会比较大，会影响启动项目，所以最好先把这个镜像拉取下来再去启动项目，拉取air+golang项目依赖的镜像命令如下

```shell
$ docker pull cosmtrek/air:latest
```

### 4、导入mysql数据

创建数据库looklook_order && 导入deploy/sql/looklook_order.sql数据

创建数据库looklook_payment && 导入deploy/sql/looklook_payment.sql数据

创建数据库looklook_travel && 导入deploy/sql/looklook_travel.sql数据

创建数据库looklook_usercenter && 导入looklook_usercenter.sql数据

### 5、启动项目

```shell
$ docker-compose up -d 
```

【注】依赖的是项目根目录下的docker-compose.yml配置

### 6、查看项目运行情况

访问 [http://127.0.0.1:9090/](https://link.segmentfault.com/?enc=VEwlTuZcn6CHcVjSnceFOw%3D%3D.%2FeJMdfpdL8KRcN2jMSQc8oj5c%2FlVC3%2B6S7MWs2Y4Rnc%3D) ， 点击上面菜单“Status”，在点击Targets ,蓝色的就是启动成了，红色就是没启动成功

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041685802.png)

【注】如果是第一次拉取项目，每个项目容器第一次构建拉取依赖，这个看网络情况，可能会比较慢有的服务，所以会导致项目启动失败或者被依赖的服务启动失败自己也启动失败了，这个很正常，如果碰到项目启动不起来的情况，比如order-api ，这时候我们去看下日志就可以

```shell
$ docker logs -f order-api 
```

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041685803.png)

很明显是因为order-rpc启动时间太久了，而order-api一直等他启动，order-rpc一定时间内没有启动成功，order-api没耐心了（超时了），就算后面order-rpc启动起来，它也不管了，这时候再去重启一次order-api就可以了，这个只是第一次创建容器会这样，之后只要不销毁容器就不会，我们去到项目根目录下重启一下

```shell
$ docker-compose restart order-api
```

【注意】一定要去到项目根目录下重启 ，因为docker-compose.yml在项目根目录

然后我们在看一下，这里我们使用docker logs 看了

```shell
  __    _   ___  
 / /\  | | | |_) 
/_/--\ |_| |_| \_ , built with Go 1.17.6

mkdir /go/src/github.com/looklook/app/order/cmd/api/tmp
watching .
watching desc
watching desc/order
watching etc
watching internal
watching internal/config
watching internal/handler
watching internal/handler/homestayOrder
watching internal/logic
watching internal/logic/homestayOrder
watching internal/svc
watching internal/types
!exclude tmp
building...
running...
```

可以看到order-api已经成功了 ，再去prometheus看一下

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041685804.png)

可以看到prometheus也显示成功了，同理把其他的也排查一次，启动成功就可以了

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041685805.png)

### 7、访问项目

由于我们使用nginx做的网关，nginx网关配置在docker-compose中，也是配置在docker-compose中，nignx对外暴露端口是8888，所以我们通过8888端口访问

```shell
$ curl  -X POST "http://127.0.0.1:8888/usercenter/v1/user/register" -H "Content-Type: application/json" -d "{\"mobile\":\"18888888888\",\"password\":\"123456\"}" 

返回:
{"code":200,"msg":"OK","data":{"accessToken":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2NzM5NjY0MjUsImlhdCI6MTY0MjQzMDQyNSwiand0VXNlcklkIjo1fQ.E5-yMF0OvNpBcfr0WyDxuTq1SRWGC3yZb9_Xpxtzlyw","accessExpire":1673966425,"refreshAfter":1658198425}}
```

【注】 如果是访问nginx失败，访问成功可以忽略，可能是nginx依赖后端服务，之前因为后端服务没启动起来，nginx这里没启动起来，重启一次nginx即可,项目根目录下重启

```shell
$ docker-compose restart nginx
```

## 六、日志收集

将项目日志收集到es（filebeat收集日志->kafka -> go-stash消费kafka日志->输出到es中, kibana查看es数据）

所以我们要提前在kafka中创建日志的topic

进入kafka容器

```shell
$ docker exec -it kafka /bin/sh
```

创建log的topic

```shell
$ cd /opt/kafka/bin
$ ./kafka-topics.sh --create --zookeeper zookeeper:2181 --replication-factor 1 -partitions 1 --topic looklook-log
```

访问kibana [http://127.0.0.1:5601/](https://link.segmentfault.com/?enc=mu4x2AQsy%2FKawux0o%2BHvMA%3D%3D.4k8OQIyV7M8tVlaz5dgTcQygJnYrE1wJFOVbOGt9jb0%3D) ， 创建日志索引

点击左上角菜单(三个横线那个东东)，找到Analytics - > 点击discover

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041685806.png)

然后在当前页面，Create index pattern->输入looklook-* -> Next Step ->选择@timestamp->Create index pattern

然后点击左上角菜单，找到Analytics->点击discover ，日志都显示了 （如果不显示，就去排查filebeat、go-stash，使用docker logs -f filebeat查看）

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041685807.png)

## 七、本项目镜像介绍

所有服务启动成功，应该是如下这些，自行对比

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041685808.png)

- nginx : 网关 （nginx->api->rpc）
- cosmtrek/air : 我们业务代码开发依赖的环境镜像，之所以用这个是因为air热加载，写代码实时编译太方便了，这个镜像是air+golang，实际上我们启我们自己的业务服务后，我们的业务服务是运行在此镜像中的
- wurstmeister/kafka ： 业务使用的kafka
- wurstmeister/zookeeper ： kafka依赖的zookeeper
- redis：业务使用的redis
- mysql: 业务使用的数据库
- prom/prometheus：监控业务
- grafana/grafana ：prometheus的ui很难看，用来显示prometheus收集来的数据
- elastic/filebeat ： 收集日志到kafka
- go-stash : 消费kafka中日志，脱敏、过滤然后输出到es
- docker.elastic.co/elasticsearch/elasticsearch ： 存储收集的日志
- docker.elastic.co/kibana/kibana ： 显示elasticsearch
- jaegertracing/jaeger-query 、jaegertracing/jaeger-collector、jaegertracing/jaeger-agent：链路追踪
- go-stash : filebeat收集日志到kafka后，go-stash去消费kafka进行数据脱敏、过滤日志中内容，最后输出到es中

## 八、项目开发建议

- app下放所有业务服务代码
- common放所有服务的公共基础库
- data项目依赖中间件产生的数据，实际开发中应该在git中忽略此目录以及此目录下产生的数据
- 生成api、rpc代码：

一般我们在生成api，rpc代码时候手动去敲goctl的命令比较长，也记不住，所以我们直接去deploy/script/gencode/gen.sh中复制代码即可。比如我在usercenter服务中新增加了一个业务，修改密码，写完api文件之后，进入到usercenter/cmd/api/desc目录下，直接复制deploy/script/gencode/gen.sh中的生成api命令运行即可

```shell
$ goctl api go -api *.api -dir ../  -style=goZero
```

生成rpc也一样，在写完proto文件后，直接粘复制deploy/script/gencode/gen.sh中的生成rpc命令运行即可

goctl >= 1.3 进入"服务/cmd/rpc/pb"目录下，执行下面命令

```shell
$ goctl rpc protoc *.proto --go_out=../ --go-grpc_out=../  --zrpc_out=../
$ sed -i "" 's/,omitempty//g' *.pb.go
```

goctl < 1.3 进入"服务/cmd"目录下，执行下面命令

```shell
$  goctl rpc proto -src rpc/pb/*.proto -dir ./rpc -style=goZero
$  sed -i "" 's/,omitempty//g'  ./rpc/pb/*.pb.go
```

【注】建议在生成rpc文件时候，在多执行一次下面那个命令，把protobuf生成的omitempty给删除掉，不然字段为nil就不返回了

- 生成kafka代码：
  因为本项目使用了go-queue的kq做消息队列，kq又依赖的kafka，实际就是使用了kafka做消息队列，但是kq默认是需要我们提前把topic建好的，不许默认自动生成，所以命令也准备好了，直接复制deploy/script/gencode/gen.sh中的创建kafka的topic代码即可

  ```shell
   kafka-topics.sh --create --zookeeper zookeeper:2181 --replication-factor 1 -partitions 1 --topic {topic}
  ```

- 生成model代码，直接运行deploy/script/mysql/genModel.sh 参数

- api项目中的.api文件我们做了拆分，统一放到每个api的desc文件夹下，因为如果所有内容都写在api中可能不便于查看，所以做了拆分，把所有方法写到一个api中，其他的实体以及req、rep统一放到一个文件夹单独定义比较清晰

- 生成model、错误处理时候使用了template重新定义，该项目用到的自定义的goctl的模版在项目data/goctl下

## 九、后续

由于项目中由于涉及到的技术栈稍微有点多，将分章节一步一步讲解，敬请关注。

# [第二篇、网关](https://segmentfault.com/a/1190000041700671)

## 1. go-zero 网关概念

go-zero架构往大的说主要由两部分组成，一个是api，一个是rpc。api主要是http对外访问的，rpc主要就是内部业务交互使用的是protobuf+grpc，当我们项目体量还不大的时候，我们可以使用api来做一个单体项目，等后续量上来之后，可以拆分到rpc做微服务，从单体转向微服务十分容易，很像java的springboot转像springcloud，非常方便。

api被很多同学理解成了网关，实际意义上来说当你的项目在使用go-zero做微服务时候，你把api当成网关也没什么大的问题，不过这样做导致的问题就是一个api对应后面多个rpc，api充当了网关，这样如果我在更新后续业务代码时候，更新任何业务都要去改动这个api网关，比如我只是改了一个小小的不起眼的服务，那就要重新构建整个api，这样不太合理，效率极低也很不方便。所以，我们只是把api当成一个聚合服务，可以拆分成多个api，比如用户服务有用户服务的rpc与api，订单服务，有订单服务的rpc与api，这样当我修改用户服务时候，我只需要更新用户的rpc与api，所有的api只是用来聚合后端rpc的业务。那有的同学就会说，我总不能每个服务解析个域名对应你的api吧，当然不能，这时候api前面就要有一个网关了，这个网关才是真正意义上的网关，比如我们常说的nginx、kong、apisix，很多微服务都内置了网关，比如springcloud提供了springcloud-gateway ， go-zero没有提供，实际也用不着单独去写一个网关，市面上的网关已经够多了，go-zero官方在晓黑板中用的nginx足够用了，当然你如果更熟悉kong、apisix都可以替换，本质上没什么不一样的，只是一个统一流量入口，统一鉴权等。

## 2. nginx网关

【注】：在看这里的时候，建议先看一下前一节的业务架构图

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041700673.jpeg)

本项目中实际也使用了nginx做为网关，使用nginx的auth_request模块作为统一鉴权，业务内部不做鉴权（设计到资产的最好业务内部做二次鉴权，主要多一层安全），nignx的网关配置在项目的data/nginx/conf.d/looklook-gateway.conf

```bash
server{
    listen 8081;
    access_log /var/log/nginx/looklook.com_access.log;
    error_log /var/log/nginx//looklook.com_error.log;

    location /auth {
        internal;
      proxy_set_header X-Original-URI $request_uri;
        proxy_pass_request_body off;
        proxy_set_header Content-Length "";
        proxy_pass http://identity-api:8001/identity/v1/verify/token;
    }

    location ~ /usercenter/ {
       auth_request /auth;
       auth_request_set $user $upstream_http_x_user;
       proxy_set_header x-user $user;

       proxy_set_header Host $http_host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header REMOTE-HOST $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_pass http://usercenter-api:8002;
   }

   location ~ /travel/ {
       auth_request /auth;
       auth_request_set $user $upstream_http_x_user;
       proxy_set_header x-user $user;

       proxy_set_header Host $http_host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header REMOTE-HOST $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_pass http://travel-api:8003;
   }


    location ~ /order/ {
       auth_request /auth;
       auth_request_set $user $upstream_http_x_user;
       proxy_set_header x-user $user;

       proxy_set_header Host $http_host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header REMOTE-HOST $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_pass http://order-api:8004;
   }

    location ~ /payment/ {
       auth_request /auth;
       auth_request_set $user $upstream_http_x_user;
       proxy_set_header x-user $user;

       proxy_set_header Host $http_host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header REMOTE-HOST $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_pass http://payment-api:8005;
   }
}
```

容器内部nginx端口是8081，使用docker暴露出去8888映射端口8081，这样外部通过8888来访问网关，使用location来匹配每个服务，当然会有人说，每加一个api服务都要来nignx配置太麻烦，你也可以使用confd统一配置，自行百度。

## 3. 举例

当我们在访问用户服务时候, [http://127.0.0.1:8888/usercen...](https://link.segmentfault.com/?enc=X1ZOE6I%2BQkovaGWJAQzohQ%3D%3D.jJj%2BU%2BGh%2FAoHkce23bRvcJa5nlA6JwlT1k%2Bh6txgUIddAiQ4E90Ih1umaqgWHwnT) , 访问了外部端口8888，然后映射到nginx内部looklook网关8081上，然后location匹配到了/usercenter/ ，在该模块开始有一行 auth_request /auth， 所以nginx不会直接去请求[http://usercenter-api:8002](https://link.segmentfault.com/?enc=4BsZruW4HxtIaa87OM5hOw%3D%3D.SxNb31slDgqnWj3VezDUPhMFCHcuYfUDLuW5K3tN7KQ%3D) ， 而是会先跳到 location /auth 模块中，auth模块会访问 [http://identity-api:8001/iden...](https://link.segmentfault.com/?enc=CxjCSDlHMM%2Fd0sSOH7jEWQ%3D%3D.QWPaof3xcOhmGTM6IB6N8znjsxv3Te0n70tMzVCZfnGvyzL1pFBANoAgrjAxoXS%2BS%2Bm9gLmCYqoO2eXBDE12jg%3D%3D); ,identity-api也是我们内部的服务，是由我们自己写的鉴权服务，实际也是用的go-zero的jwt

进入identity-api 只做了2件事情（具体可以看looklook项目中的identity-api代码）

1、判断当前访问的路由（usercenter/v1/user/detail ）是否需要登录。这里的路由是否需要登录，可以在identity-api中配置，代码已经实现好了。

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041700674.png)

2、解析传递的token到header中

- 如果当前访问的路由需要登录：
  - token解析失败：就会返回给前端http401错误码；
  - token解析成功：就会将解析出来的userId放入header的x-user中返回给auth模块，auth模块会把header传递给对应访问的服务（usercenter）, 这样我们在usercenter直接就可以拿到该登录用户的id了
- 如果当前访问的路由不需要登录：
  - 前端header中传递了token
    - 如果token校验失败：返回http401；
    - 如果token校验成功：就会将解析出来的userId放入header的x-user中返回给auth模块，auth模块会把header传递给对应访问的服务（usercenter）, 这样我们在usercenter直接就可以拿到该登录用户的id了
  - 前端header中没传递token：userid 会传递 0 给后端服务

## 4、总结

这样我们就可以统一入口，统一鉴权，也可以统一收集日志上报，用作错误分析，或者访问用户的行为分析。因为我们日常对nginx用的比较多，也比较熟悉，如果各位同学对kong、apisix比较熟悉，在了解了上方go-zero使用网关的概念就可以直接替换也是一样的。

# [第三篇、鉴权](https://segmentfault.com/a/1190000041719083)

## 1、鉴权服务

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041719085.jpeg)

### 1.1 identity-api

identity主要是用来做鉴权服务的，前面我们nginx网关的时候有提到。在访问一个资源的时候，nginx内部会先来identity-api中解析token，identity-api会去请求identity-rpc，所有的验证与颁发token，统一是在identity-rpc中做的

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041719086.png)

我们会从header的Authorization中获取token，从x-Original-Uri获取访问的资源路径

- 如果当前访问的路由需要登录：
  - token解析失败：就会返回给前端http401错误码；
  - token解析成功：就会将解析出来的userId放入header的x-user中返回给auth模块，auth模块会把header传递给对应访问的服务（usercenter）, 这样我们在usercenter直接就可以拿到该登录用户的id了
- 如果当前访问的路由不需要登录：
  - 前端header中传递了token
    - 如果token校验失败：返回http401；
    - 如果token校验成功：就会将解析出来的userId放入header的x-user中返回给auth模块，auth模块会把header传递给对应访问的服务（usercenter）, 这样我们在usercenter直接就可以拿到该登录用户的id了
  - 前端header中没传递token：userid 会传递 0 给后端服务

urlNoAuth方法判断当前资源是否在yml中配置可以不登录

```go
//当前url是否需要授权验证
func (l *TokenLogic) urlNoAuth(path string) bool {
   for _, val := range l.svcCtx.Config.NoAuthUrls {
      if val == path {
         return true
      }
   }
   return false
}
```

isPass方法就是去identity-rpc校验token，主要也是使用了go-zero的jwt的方法

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041719087.png)

### 1.2 identity-rpc

当我们在注册、登录成功时候，用户服务会调用identity-rpc生成token，所以我们统一在identity-rpc中颁发、校验token，这样就不用每个服务都要写个jwt去维护。

当identity-api请求进来时候，identity-api自己可以解析出来userid，但是我们要检验这个token是否是过期，就要去后端rpc中的redis中去进行二次校验（当然如果你觉得这里多一次请求，你可以把这一步放到api里直接请求redis也可以），经过rpc的validateToken方法校验

```protobuf
message ValidateTokenReq {
  int64 userId = 1;
  string token = 2;
}
message ValidateTokenResp {
  bool ok = 1;
}

rpc validateToken(ValidateTokenReq) returns(ValidateTokenResp);
```

校验之前登录、注册等授权时候颁发出去存在redis的token是否正确、过期。

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041719088.png)

这样api就可以返回给nginx的auth模块是否失败，如果失败auth会直接返回给前端http code 401（所以你们前端应该是先判断http状态码>=400全部异常，再判断业务错误码） , 如果成功直接访问后端服务了拿到数据直接返回给前端展示

## 2、安装goctl 与 protoc、protoc-gen-go

【注】这个跟鉴权没什么关系，只是后面写代码要用到，在这里最好给安装了

### 2.1 安装goctl

```shell
# for Go 1.15 and earlier
GO111MODULE=on GOPROXY=https://goproxy.cn/,direct go get -u github.com/zeromicro/go-zero/tools/goctl@latest

# for Go 1.16 and later
GOPROXY=https://goproxy.cn/,direct go install github.com/zeromicro/go-zero/tools/goctl@latest
```

验证是否安装成功

```shell
$ goctl --version
```

Goctl自定义模版template：将项目目录下的data/goctl文件夹内容copy到 HOME 目录的 .goctl 中，goctl在生成代码时候会优先根据这个模版下内容生成

```shell
$ cp -r data/goctl ~/.goctl
```

### 2.2 安装protoc

链接：[https://github.com/protocolbu...](https://link.segmentfault.com/?enc=PTJs7MKQyK72I1YNdt9qvg%3D%3D.V62kC3a1BF5jnjYLk8Vnt4xTQohwz9j7zaQTiXu%2BhB3Tc%2B73qBlSrCXQgE3Ld9TXMAU9w0FdSpVqVbEAw1C08g%3D%3D)

直接找到对应平台的protoc，我是mac intel芯片，所以直接找到 protoc-3.19.3-osx-x86_64.zip ，解压出来后进入该目录下的bin目录中，将protoc直接copy到你的gopath/bin目录下即可。

验证是否安装成功

```shell
$ protoc --version
```

### 2.3 安装protoc-gen-go

```shell
$ GOPROXY=https://goproxy.cn/,direct go install google.golang.org/protobuf/cmd/protoc-gen-go@latest 
```

查看$GOPATH/bin下是否有protoc-gen-go即可

【注】：如果后续在使用goctl生成代码时候，遇到以下问题

```shell
protoc  --proto_path=/Users/seven/Developer/goenv/go-zero-looklook/app/usercenter/cmd/rpc/pb usercenter.proto --go_out=plugins=grpc:/Users/seven/Developer/goenv/go-zero-looklook/app/usercenter/cmd/rpc --go_opt=Musercenter.proto=././pb
goctl: generation error: unsupported plugin protoc-gen-go which installed from the following source:
google.golang.org/protobuf/cmd/protoc-gen-go, 
github.com/protocolbuffers/protobuf-go/cmd/protoc-gen-go;

Please replace it by the following command, we recommend to use version before v1.3.5:
go get -u github.com/golang/protobuf/protoc-gen-go
goctl version: 1.3.0 darwin/amd64
```

直接执行

```shell
$ GOPROXY=https://goproxy.cn/,direct go get -u github.com/golang/protobuf/protoc-gen-go
```

### 2.4 安装protoc-gen-go-grpc

```shell
$ GOPROXY=https://goproxy.cn/,direct go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

## 3、总结

总的来说，identity还算是比较简单的，整个流程如下：

用户发起请求资源 -> nginx网关->匹配到对应服务模块 -> auth模块->identity-api ->identity-rpc -> 用户请求的资源

# [第四篇、用户中心](https://segmentfault.com/a/1190000041722973)

## 一、用户中心业务架构图

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041722975.jpeg)

## 二、依赖关系

usercenter-api（用户中心api） 依赖 identity-rpc（授权认证rpc）、usercenter-rpc（用户中心rpc）

usercenter-rpc（用户中心rpc）依赖 identity-rpc（授权中心rpc）

我们看项目usercenter/cmd/api/desc/usercenter.api ，所有的用户api对外的http方法都在这里面

这里面有4个业务注册、登录、获取用户信息、微信小程序授权

## 三、注册举例

### 1、注册api服务

我们在写api服务代码的时候是先要在usercenter.api中定义好service中的方法，然后在desc/user中写request、response，这样拆分开的好处是不那么臃肿

a、在usercenter.api中定义注册方法如下

```go
// 用户模块v1版本的接口
@server(
    prefix: usercenter/v1
    group: user
)
service usercenter {
  @doc "注册"
    @handler register
    post /user/register (RegisterReq) returns (RegisterResp)
  
  .....
}
```

b、在app/usercenter/cmd/api/desc/user/user.api中定义RegisterReq\RegisterResp

```go
type (
    RegisterReq {
        Mobile   string `json:"mobile"`
        Password string `json:"password"`
    }
    RegisterResp {
        AccessToken  string `json:"accessToken"`
        AccessExpire int64  `json:"accessExpire"`
        RefreshAfter int64  `json:"refreshAfter"`
    }
)
```

c、goctl生成api代码

1）命令行进入app/usercenter/cmd/api/desc目录下。

2）去项目目录下deploy/script/gencode/gen.sh中，复制如下一条命令，在命令行中执行（命令行要切换到app/usercenter/cmd目录）

```shell
$ goctl api go -api *.api -dir ../  -style=goZero
```

d、打开app/usercenter/cmd/api/internal/logic/user/register.go文件

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041722976.png)

这里就很容易了，直接调用user的rpc服务即可

这里有个小技巧，很多同学感觉rpc服务返回的字段跟api定义差不多，每次都要手动去复制很麻烦，那么go有没有像java一样的BeanCopyUtils.copy 这种工具呢？答案肯定是有的，可以看上面的代码copier.Copy ，这个库是gorm作者的另一款新作，是不是很兴奋。 那我们继续看看调用后端的rpc是什么样子的。

### 2、注册rpc服务

- 定义protobuf文件

我们在app/usercenter/cmd/rpc/pb中新建usercenter.proto，写入注册方法

```protobuf
// req 、resp
message RegisterReq {
  string mobile = 1;
  string nickname = 2;
  string password = 3;
  string authKey = 4;
  string authType = 5;
}

message RegisterResp {
  string accessToken = 1;
  int64  accessExpire = 2;
  int64  refreshAfter = 3;
}

// service
service usercenter {
  rpc register(RegisterReq) returns(RegisterResp);
  ...
}
```

- 使用goctl生成代码，这里不需要自己手动敲

1）命令行进入app/usercenter/cmd/rpc/pb目录下。

2）去项目目录下deploy/script/gencode/gen.sh中，复制如下两条命令，在命令行中执行（命令行要切换到app/usercenter/cmd目录）

```shell
$  goctl rpc protoc *.proto --go_out=../ --go-grpc_out=../  --zrpc_out=../
$  sed -i "" 's/,omitempty//g' *.pb.go
```

- 打开app/usercenter/cmd/rpc/internal/logic/registerLogic.go写逻辑代码

  ![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041722977.png)

注册设计到2张表，一个user表，一个user_auth表，user是存储用户基本信息的，user_auth是可以根据不同平台授权登录的相关信息，所以这里设计到本地事务，由于go-zero的事务要在model中才能使用，但是我在model中做了个处理，把它在model中暴露出来，就可以在logic中使用

model中定义了Trans方法暴露事务给logic

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041722978.png)

在logic中直接使用

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041722979.png)

由于项目支持小程序、手机号，小程序注册不需要密码，所以在处理密码时候做了个处理，手机号注册就要传递密码，小程序注册就不需要传递密码，至于手机号注册密码不能为空要在手机号注册时候的api服务自己判断

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041722980.png)

在usercenter-rpc注册成功之后，需要请求token给前端登录，直接请求identity-rpc颁发该用户的token

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041722981.png)

identity-rpc中如下

```protobuf
message GenerateTokenReq {
  int64 userId = 1;
}
message GenerateTokenResp {
  string accessToken = 1;
  int64  accessExpire = 2;
  int64  refreshAfter = 3;
}


service identity{
  //生成token，只针对用户服务开放访问
  rpc generateToken(GenerateTokenReq) returns(GenerateTokenResp);
  .....
}
```

generatetokenlogic.go

```go
// GenerateToken 生成token，只针对用户服务开放访问.
func (l *GenerateTokenLogic) GenerateToken(in *pb.GenerateTokenReq) (*pb.GenerateTokenResp, error) {

    now := time.Now().Unix()
    accessExpire := l.svcCtx.Config.JwtAuth.AccessExpire
    accessToken, err := l.getJwtToken(l.svcCtx.Config.JwtAuth.AccessSecret, now, accessExpire, in.UserId)
    if err != nil {
        return nil, errors.Wrapf(ErrGenerateTokenError, "getJwtToken err userId:%d , err:%v", in.UserId, err)
    }

    //存入redis
    userTokenKey := fmt.Sprintf(globalkey.CacheUserTokenKey, in.UserId)
    err = l.svcCtx.RedisClient.Setex(userTokenKey, accessToken, int(accessExpire))
    if err != nil {
        return nil, errors.Wrapf(ErrGenerateTokenError, "SetnxEx err userId:%d, err:%v", in.UserId, err)
    }

    return &pb.GenerateTokenResp{
        AccessToken:  accessToken,
        AccessExpire: now + accessExpire,
        RefreshAfter: now + accessExpire/2,
    }, nil
}
```

注册成功并去identity-rpc拿到token、token过期时间、置换token的时间给api服务

## 四、业务获取登录用户id

当我们在获取用户信息，或者下单等场景下总要获取登录用户的id，前一篇我们讲到，我们在授权identity服务中校验完token，解析出来的userId会放到header中返回给nginx的authReuest

在文件app/identity/cmd/api/internal/handler/verify/tokenHandler.go

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041722982.png)

nginx通过authRequest然后访问后端的服务时候，会把header内容传递给后端服务，因为我们在nginx中配置了如下

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041722983.png)

那这样的话，我们在后端服务就可以拿到这个userId了，比如我们现在访问usercenter/v1/user/detail获取当前登录用户信息

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041722984.png)

ok，可以看到我们通过 ctxdata.GetUidFromCtx(l.ctx)就可以拿到，为什么这么神奇呢？我们点开看看这个方法

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041722985.png)

实际上就是从ctx中拿到的userId，是不是很奇怪，我们明明在nignx就放在了header中，你在go的业务代码中为什么能通过ctx拿到？

### 1、【小技巧】middleware

当nginx在header中携带了x-user就是userId来访问后端服务的时候，我们后端服务在启动时main函数会加载一个全局中间件，比如usercenter-api中的main

app/usercenter/cmd/api/usercenter.go

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041722986.png)

这里定义了全局中间件，只要有请求到我们usercenter-ap某个方法之前，都会先进入全局中间件中，中间件具体内容如下

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041722987.png)

所以是不是一下就明白了，在请求我们usercenter/v1/user/detail时候，会先进入这个中间件，在这个中间件内，我们通过nginx的header中的X-User拿到解析后的userId放到ctx中，那继续进入到usercenter/v1/user/detail时候，我们是不是就可以通过ctx直接取出来在业务中用啦，一切真相大白。

同样其他用户中心服务登录、获取登录用户信息、小程序授权登录都是一个道理，这里就不再啰嗦了，自行看代码即可

【注】小程序授权登录，记得修改配置文件，这里的配置文件是假的，改成自己的

# [第五篇、民宿服务](https://segmentfault.com/a/1190000041730014)

## 1、民宿服务业务架构图

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041730016.png)

## 2、依赖关系

travel-api（民宿api） 依赖 travel-rpc（民宿rpc）、usercenter-rpc（用户中心rpc）

usercenter-rpc（用户中心rpc）依赖 identity-rpc（授权中心rpc）

travel分为几个业务

- homestay ：民宿房源

```protobuf
// 民宿模块v1版本的接口
@server(
    prefix: travel/v1
    group: homestay
)
service travel {
    @doc "民宿列表（为你优选）"
    @handler homestayList
    post /homestay/homestayList (HomestayListReq) returns (HomestayListResp)
    
    @doc "房东所有民宿列表"
    @handler businessList
    post /homestay/businessList (BusinessListReq) returns (BusinessListResp)
    
    @doc "猜你喜欢民宿列表"
    @handler guessList
    post /homestay/guessList (GuessListReq) returns (GuessListResp)
    
    @doc "民宿详情"
    @handler homestayDetail
    post /homestay/homestayDetail (HomestayDetailReq) returns (HomestayDetailResp)
}
```

- homestayBusiness ： 民宿店家

```protobuf
// 店铺模块v1版本的接口
@server(
    prefix: travel/v1
    group: homestayBussiness
)
service travel {
    @doc "最佳房东"
    @handler goodBoss
    post /homestayBussiness/goodBoss (GoodBossReq) returns (GoodBossResp)
    
    @doc "店铺列表"
    @handler homestayBussinessList
    post /homestayBussiness/homestayBussinessList (HomestayBussinessListReq) returns (HomestayBussinessListResp)
    
    @doc "房东信息"
    @handler homestayBussinessDetail
    post /homestayBussiness/homestayBussinessDetail (HomestayBussinessDetailReq) returns (HomestayBussinessDetailResp)
}
```

- homestayComment ： 民宿评论

```protobuf
// 民宿评论模块v1版本的接口
@server(
    prefix: travel/v1
    group: homestayComment
)
service travel {
    @doc "民宿评论列表"
    @handler commentList
    post /homestayComment/commentList (CommentListReq) returns (CommentListResp)
}
```

## 3、举例：民宿列表（为你优选）

### 1、api服务

1、写api接口文件

app/travel/cmd/api/desc/homestay/homestay.api

```protobuf
type (
    HomestayListReq {
        LastId   int64  `json:"lastId"`
        PageSize int64  `json:"pageSize"`
        RowType  string `json:"rowType"` //preferredHomestay:优选民宿
    }
    HomestayListResp {
        List []Homestay `json:"list"`
    }
)
```

app/travel/cmd/api/desc/travel.api

```protobuf
import (
    "homestay/homestay.api"
    ....
)

// 民宿模块v1版本的接口
@server(
    prefix: travel/v1
    group: homestay
)
service travel {
    @doc "民宿列表（为你优选）"
    @handler homestayList
    post /homestay/homestayList (HomestayListReq) returns (HomestayListResp)
    ......
}
```

2、goctl生成api代码

1）命令行进入app/travel/cmd/api/desc目录下。

2）去项目目录下deploy/script/gencode/gen.sh中，复制如下一条命令，在命令行中执行（命令行要切换到app/travel/cmd目录）

```shell
$ goctl api go -api *.api -dir ../  -style=goZero
```

3、打开app/travel/cmd/api/internal/logic/homestay/homestayListLogic.go

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041730017.png)

因为我们的推荐是在后台配置的，所以我们创建了一个活动表（这里你也可以选择配置到redis中），总之我们就是先从活动表中拿到配置的推荐民宿id，然后再通过id去获取对应民宿信息列表。

### 2【小技巧】 mapreduce

这里可以看到，我拿到了id集合之后，不是普通的foreach一个个获取，而是使用了go-zero为我们封装好了的mapreduce获取数据，这样就可以并发去获取数据，而不是要去取一个完成之后再取下一个，时间上大大缩短了，这里只是想给搭建展示这样一个功能，有的同学非要较真，可以传递一个id slice或者id arr到rpc，然后在rpc中在去并发获取每个，这样也没什么不好，我这里只是给大家展示这个功能

### 3、rpc服务

定义protobuf文件

app/travel/cmd/rpc/pb/travel.proto

```protobuf
// model
message Homestay {
    int64   id = 1;
    string  title = 2;
    string  subTitle = 3;
    string  banner = 4;
    string  info = 5;
    int64   peopleNum = 6;            // 容纳人的数量
    int64   homestayBusinessId = 7;   // 店铺id
    int64   userId = 8;               // 房东id
    int64   rowState = 9;             // 0:下架 1:上架
    int64   rowType = 10;             // 售卖类型0：按房间出售 1:按人次出售
    string  foodInfo = 11;            // 餐食标准
    int64   foodPrice = 12;           // 餐食价格(分)
    int64   homestayPrice = 13;       // 民宿价格(分)
    int64   marketHomestayPrice = 14; // 民宿市场价格(分)
}

// req 、resp
message HomestayDetailReq {
  int64   id = 1;
}
message HomestayDetailResp {
  Homestay homestay = 1;
}

// service
service travel {
    // 民宿详情
    rpc homestayDetail(HomestayDetailReq) returns(HomestayDetailResp);
}
```

- 使用goctl生成代码，这里不需要自己手动敲

  1）命令行进入app/travel/cmd/rpc/pb目录下。

  2）去项目目录下deploy/script/gencode/gen.sh中，复制如下两条命令，在命令行中执行（命令行要切换到app/travel/cmd目录）

  ```shell
  $ goctl rpc protoc *.proto --go_out=../ --go-grpc_out=../  --zrpc_out=../
  $ sed -i "" 's/,omitempty//g' *.pb.go
  ```

- 打开app/travel/cmd/rpc/internal/logic/homestayDetailLogic.go写逻辑代码

  ![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041730018.png)

  这里没什么逻辑，查询Findone，然后返回给api，因为api那边是通过id传递过来的，然后可以看到我们这边又一次使用了前一章提到的gorm作者提供的另外一款神器copier，上一节是在api中使用，将rpc的proto文件的数据copy到api文件 ， 这里可以看到，我们把model返回的数据copy给proto的数据同样可以用，怎么样是不是很方便。

### 4、【小技巧】 model cache、singleflight

在这里为什么我们不去findlist，是因为我们在findone方法中有缓存，我们一个个根据id查询数据时候，只有第一次会命中db，其他时间基本都是命中的redis cache，这样不仅速度快，就算流量激增的时候，也不会全部打到db上，而是都在redis上，这样会大大提高我们系统的访问速度以及db支撑能力。

一般我们自己维护db cache会写的零零散散，但是go-zero使用了配套内置工具goctl生成的model，自带sqlc+sqlx实现的代码，实现了自动缓存管理，我们根本不需要去管理缓存，只需要用sqlx写 sql数据，sqlc会自动帮我们管理缓存，并且是通过**singleflight** ,也就是说即使缓存在某个时间失效，在失效那一刻同时有大量并发请求进来时，go-zero在查询db时候也只会放行一个线程进来，其他线程是在等待，当这个线程从数据库拿数据回来之后将该数据缓存到redis同时所有之前等待线程共享此数据返回，后续在进来的线程查相同数据时，就只会进入到redis中而不会进入到db。

这样rpc拿到所有数据之后，就可以返回给前端显示了。

## 4、小结

其他的几个服务没有业务什么逻辑性的这里就不再一一说明，看api文档基本都知道是什么了，根据上面例子代码自行查看即可，后面有牵扯业务复杂的地方会逐一说明

# [第六篇、订单服务](https://segmentfault.com/a/1190000041749029)

## 1、订单服务业务架构图

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041749031.jpeg)

## 2、依赖关系

order-api（订单api）

- order-rpc（订单rpc）
- payment-rpc（支付rpc）

payment-rpc（支付rpc）

- mqueue-rpc(消息队列)

order-rpc（订单rpc）

- mqueue-rpc(消息队列)
- travel-rpc（民宿rpc）

## 3、订单举例

### 3.1 下订单

**1、用户在去浏览travel服务中的民宿homestay看中选择日期之后要下单，调用下单api接口**

app/order/cmd/api/desc/order.api

```protobuf
// 订单模块v1版本的接口
@server(
   prefix: order/v1
   group: homestayOrder
)
service order {
   @doc "创建民宿订单"
   @handler createHomestayOrder
   post /homestayOrder/createHomestayOrder (CreateHomestayOrderReq) returns (CreateHomestayOrderResp)
   
   .....
}
```

**2、order-api中调用order-rpc**

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041749032.png)

**3、rpc中校验条件创建订单之后 ，会调用mqueue-rpc创建一个延迟关闭订单的消息队列**

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041749033.png)

**4、mqueue-rpc 延迟队列**

延迟队列是用的asynq，asynq是基于redis的高性能队列，同时支持消息队列、定时队列、固定周期队列，但是我们这个项目为了演示go-zero官方的消息队列go-queue（go-queue是基于kafka的），所以消息队列用的go-queue，延迟队列、定时任务用asynq。这里注意，这里只是往延迟队列添加延迟任务，具体执行不在这里，那我们去看看20分钟之后具体执行的代码，在app/order/cmd/mq

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041749034.jpeg)

**5、延迟20分钟执行的任务队列**

在app/order/cmd/mq，这里我要说明一下，go-zero官方goctl支持生成的服务目前是api、rpc，目前暂时还没有支持console、mq等，但是go-zero提供了ServiceGroup，方便我们管理我们自己任何的服务，所以在mq中我是用了ServiceGroup管理了服务，这也是官方推荐的使用方式，代码如下:

1）app/order/cmd/mq/order.go 首先我们看main.go

```go
func main() {
    flag.Parse()
    var c config.Config

    conf.MustLoad(*configFile, &c)
    prometheus.StartAgent(c.Prometheus)

    serviceGroup := service.NewServiceGroup()
    defer serviceGroup.Stop()

    for _, mq := range listen.Mqs(c) {
        serviceGroup.Add(mq)
    }
    serviceGroup.Start()
}
```

`ServiceGroup` 可以添加任何service进去，但是如何成为一个service呢？ 那你就要实现两个方法一个Starter、一个Stoper

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041749035.png)

2）我们在main中可以看到循环listen.Mqs(c) ，那我们看看listen.Mqs(c) 都有哪些

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041749036.png)

我们不止要监听asynq的延迟队列、定时队列，后面我们还想监听go-queue的kafka消息队列，在代码上我们考虑不想把go-queue的kafka消息队列跟asynq的延迟队列、定时队列放到一起去，所以这里做了个分类

3）asyny的延迟消息队列

定义asynq

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041749037.png)

定义路由

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041749038.png)

具体实现逻辑（关闭订单逻辑）

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041749039.png)

所以我们启动这个order-mq的时候，asynq就会被加载、定义路由，当我们之前添加的延迟队列到了20分钟，就会自动执行关闭订单逻辑，如果订单没有被支付，这里就会关闭掉订单，支付了就忽略掉了，这样就可以不用使用定时任务轮训关闭订单了，哈哈

### 3.2 订单列表

没啥逻辑，就是查出来显示罢了，自己看一下就好了

```protobuf
// 订单模块v1版本的接口
@server(
   prefix: order/v1
   group: homestayOrder
)
service order {

   @doc "用户订单列表"
   @handler userHomestayOrderList
   post /homestayOrder/userHomestayOrderList (UserHomestayOrderListReq) returns (UserHomestayOrderListResp)
   
}
```

### 3.3 订单明细

没啥逻辑，就是查出来显示罢了，自己看一下就好了

```protobuf
// 订单模块v1版本的接口
@server(
    prefix: order/v1
    group: homestayOrder
)
service order {

    @doc "用户订单明细"
    @handler userHomestayOrderDetail
    post /homestayOrder/userHomestayOrderDetail (UserHomestayOrderDetailReq) returns (UserHomestayOrderDetailResp)
}
```

## 4、结尾

下了订单，当然我们要支付了，那就看下一篇支付服务吧

# [第七篇、支付服务](https://segmentfault.com/a/1190000041760701)

## 1、支付服务业务架构图

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041760703.png)

## 2、依赖关系

payment-api（支付api）

- order-rpc（订单rpc）
- payment-rpc（支付rpc）
- usercenter（用户rpc）

payment-rpc（支付rpc）

- mqueue-rpc(消息队列)

order-rpc（订单rpc）

- mqueue-rpc(消息队列)
- travel-rpc

usercenter（用户rpc）

- identity-rpc(授权认证rpc)

## 3、微信支付举例

### 3.1 创建支付预处理订单

1、用户在我们这边创建完订单之后，要去微信那边创建预支付订单

app/payment/cmd/api/desc/payment.api

```less
// 支付服务v1版本的接口
@server(
    prefix: payment/v1
    group: thirdPayment
)
service payment {
    @doc "第三方支付：微信支付"
    @handler thirdPaymentwxPay
    post /thirdPayment/thirdPaymentWxPay (ThirdPaymentWxPayReq) returns (ThirdPaymentWxPayResp)
    
    ...
}
```

app/payment/cmd/api/internal/logic/thirdPayment/thirdPaymentwxPayLogic.go

- ThirdPaymentwxPay

见下图，我们创建微信预支付订单时候做了一次封装，因为我们平台后续支付业务肯定不止民宿支付订单，肯定还会有其他的，比如我们后续可以推出商城，推出课程等，所以在这里使用switch做了个业务分类，目前我们只有民宿订单，但是除了查询业务不一样，其他都一样，我们把一样的逻辑封装起来，所以我们继续看封装后的方法 `createWxPrePayOrder`

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041760704.png)

app/payment/cmd/api/internal/logic/thirdPayment/thirdPaymentwxPayLogic.go

- createWxPrePayOrder

这里就是拿到用户的登陆userId去换openid（这块我们之前注册登陆那里有小程序注册登陆，那时候就获取了openid），然后调用paymentRpc中的CreatePayment创建我们本地的支付流水单号，再通过调用微信sdk-> svc.NewWxPayClientV3(这里是我基于go-zero封装了一次，没啥难度都能看懂) ，

然后在微信端创建了一个关联我们本地流水单号的预支付订单，返回给前端，前端通过js发起请求即可

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041760705.png)

### 3.2 微信支付回调

当前端拿着我们给的微信预处理订单发起支付，用户输入密码支付成功后，微信服务器会回调我们服务器，回调地址在我们配置中填写的

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041760706.png)

这个回调地址，一定要填写我们支付api服务中的回调处理方法,也就是如下图的接口，这样我们才能接收到微信回调进来，我们才可以做后续处理。

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041760707.png)

微信回调回来之后，我们要处理回调逻辑，我们要调用verifyAndUpdateState 将我们流水单号改为已支付

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041760708.png)

我们来看看verifyAndUpdateState方法，我们要查询单号是否存在，比对回调回来的金额与创建时候金额是否一致更新流水单号即可。这里不用在校验签名了，前一步的sdk已经做了处理了

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041760709.png)

这里还要给前端写一个轮训接口，前端用户支付成功后前端不能以前端的微信返回结果为准，要通过后端提供的接口轮训，判断这个流水单是否真的是后端返回支付成功状态，如果这个接口返回成功才算成功，微信前端返回的不能作为依据，因为微信前端返回的不安全，一般开发都明白不知道的自己百度。

### 3.3 支付成功发送小程序模版消息

我们支付回调成功之后，会给用户发送一个入驻码，去了商家那里要展示这个码，商家通过后台核对码，其实就是美团的样子，我们去美团下单，美团会给你个码，用户拿着这个码去入住或者消费等。

ok，回调成功，我们会调用pyamentRpc去修改当前流水单状态成功

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041760710.png)

我们来看看paymentRpc中做了什么，

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041760711.png)

前面是校验，核心做了两件事情，第一是更新状态，第二向消息队列发送了一条消息，我们看看消息队列中对应的代码

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041760712.png)

可以看到我们使用了go-queue发送了一条kq消息到kafka，而不是asynq延迟消息，因为我们想让所有订阅了该支付状态的业务都能收到此消息后做相应的处理，虽然目前我们只有一个地方监听做处理（发送小程序模版消息通知用户支付成功），所以这里就是发了一条该支付流水相关信息到kafka中，这里跟之前订单那里是一样的只是添加消息到队列，没有处理，那我们看看order-mq中怎么处理的。

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041760713.png)

前面order一节已经介绍了整个order-mq的运作机制，这里不再多说了，我们只说kq这里

当order-mq启动后，go-queue会监听kafka中的消息

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041760714.png)

我们再来看下具体实现 , 当前面支付回调成功添加到kafka中时候，order-mq中kafka会接受消息，也就是PaymentUpdateStatusMq.Consume会接收到kafka的消息，然后反序列化数据，传递给execService 执行具体业务，那execService中执行了什么呢？

可以看到下方红框内，一个是修改订单状态（非支付状态，订单也有自己状态），一个是发消息（短信、微信小程序模版消息）给用户

app/order/cmd/mq/internal/mqs/kq/paymentUpdateStatus.go

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041760715.png)

修改订单状态的我们就不看了，我们可以来看看发送小程序模版消息，下方LiveStateDate\LiveEndDate之前调试写死的，这个直接改成方法传递过来的时间就好了，转换一下

【注】用户想收到小程序模版消息，必须在前端让用户授权才行，这是小程序必须的不是我们能控制的

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041760716.png)

这里发送消息我们也不是真正的调用微信sdk去发送消息，也是往消息队列MqueueRpc中插入模版消息（其实这里也可以直接发），然后由message消息服务从kafka中取出来真正发送，是想所有的短信、email、微信等消息统一从这个服务发送出去，这个自己根据自己公司业务或者架构去灵活设计吧，不一定非得这样。

那我们说到这里了就直接去看看message消息服务代码吧

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041760717.png)

message业务中只有一个mq，因为他不需要rpc、api，只需要定时从队列去消息发送消息，所以它运行逻辑跟order-mq一样的，同样适用serviceGroup管理

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041760718.png)

我们不细说了，运行逻辑可以去看订单服务那一节的order-mq有细说，我们只看具体实现逻辑,go-queue从kafka队列中取出每一条要发送的微信小程序模版消息数据，然后反序列化交给execService去处理，我们来看看execService

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041760719.png)

execService 主要就是整合数据，通过小程序sdk的client 发送给小程序即可，这里有个注意点，小程序是可以区分环境的，是发送到线上小程序还是体验版小程序，在下方红色框内有做区分，实际这样是不安全的 通过这种方式，最好搞到配置文件里，万一开发环境有人捣乱改成formal，随便发给别人openid就出事了，这个自己可以改改哈

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041760720.png)

## 4、小结

到这里基本上整体项目服务逻辑都差不多说明完了，后续会介绍收集日志、监控、部署等

# [第八篇、各种队列](https://segmentfault.com/a/1190000041765889)

## 1、概述

消息队列有很多种，有rabbitmq、rocketmq、kafka等常用的，其中go-queue([https://github.com/zeromicro/...](https://link.segmentfault.com/?enc=xHgdZbjpiEKjwqb3yEf1tw%3D%3D.HoaAFwi8cY1QbwoMNyHg21UYygQDtzrjDyx8cmrVpMqk0dgt%2B4aKuysyJrMtfx9a))是go-zero官方开发的消息队列组件，其中分为2类，一种是kq、一种是dq，kq是基于kafka的消息队列，dq是基于beanstalkd的延迟队列，但是go-queue不支持定时任务。具体想更多了解go-queue的我之前也写过一篇教程可以去看一下这里不细说了。

本项目采用的是go-queue做消息队列，asynq做延迟队列、定时队列

为什么使用asynq的几个原因

- 直接基于redis，一般项目都有redis，而asynq本身就是基于redis所以可以少维护一个中间件
- 支持消息队列、延迟队列、定时任务调度 ， 因为希望项目支持定时任务而asynq直接就支持
- 有webui界面，每个任务都可以暂停、归档、通过ui界面查看成功失败、监控

为什么asynq支持消息队列还在使用go-queue？

- kafka的吞吐是业绩出名的，如果前期量不大可以直接用asynq
- 没啥目的，就是想给你们演示一下go-queue

在我们使用go-zero的时候，goctl给我们带了很大的便利，但是目前go-zero只有生成api、rpc，很多同学在群里问定时任务、延迟队列、消息队列如何生成，目录结构该怎样做，其实go-zero是为我们设计好了的，就是serviceGroup，使用serviceGroup管理你的服务。

## 2、如何使用

在前面订单、消息等场景我们其实已经演示过了，这里再额外单独补充一次

我们还是拿order-mq来举例子，显然使用goctl生成api、rpc不是我们想要的，那我们就自己使用serviceGroup改造，目录结构还是延续api的基本差不多，只是将handler改成了listen ， 将logic换成了mqs。

### 2.1 在main中代码如下

```go
var configFile = flag.String("f", "etc/order.yaml", "Specify the config file")

func main() {
    flag.Parse()
    var c config.Config

    conf.MustLoad(*configFile, &c)
    // log, prometheus, trace, metricsUrl
    if err := c.SetUp(); err != nil {
        panic(err)
    }

    serviceGroup := service.NewServiceGroup()
    defer serviceGroup.Stop()

    for _, mq := range listen.Mqs(c) {
        serviceGroup.Add(mq)
    }

    serviceGroup.Start()
}
```

- 首先我们要定义配置以及解析配置。

- 其次为什么我们要在这里加SetUp而api、rpc不需要呢？因为api、rpc都是在MustNewServer中已经框架写的，但是我们用serviceGroup管理没有,可以手动点进去SetUp看看，这个方法中包含了log、prometheus、trace、metricsUrl的定义，一个方法可以省很多事情，这样我们直接修改配置文件就可以实现日志、监控、链路追踪了。

- 接下来就是go-zero的serivceGroup管理服务了，serviceGroup是用来管理一组service的，那service其实就是一个接口，代码如下

  Service (代码在go-zero/core/service/servicegroup.go)

  ```go
  // Service is the interface that groups Start and Stop methods.
  Service interface {
      Starter // Start
      Stopper // Stop
  }
  ```

  所以，只要你的服务实现了这两个接口，就可以加入到serviceGroup统一管理

  那可以看到我们把所有的mq都实现这个接口，然后统一放到都 list.Mqs中，在启动服务即可

### 2.2 mq分类管理

go-zero-looklook/app/order/cmd/mq/internal/listen目录下代码

该目录下代码是统一管理不同类型mq，因为我们要管理kq、asynq可能后续还有rabbitmq、rocketmq等等，所以在这里做了分类方便维护

统一管理在go-zero-looklook/app/order/cmd/mq/internal/listen/listen.go,然后在main中调用listen.Mqs可以获取所有mq一起start

```go
// 返回所有消费者
func Mqs(c config.Config) []service.Service {
    svcContext := svc.NewServiceContext(c)
    ctx := context.Background()

    var services []service.Service

    // kq ：消息队列.
    services = append(services, KqMqs(c, ctx, svcContext)...)
    // asynq：延迟队列、定时任务
    services = append(services, AsynqMqs(c, ctx, svcContext)...)
    // other mq ....

    return services
}
```

go-zero-looklook/app/order/cmd/mq/internal/listen/asynqMqs.go就是定义的asynq

```go
// asynq
// 定时任务、延迟任务
func AsynqMqs(c config.Config, ctx context.Context, svcContext *svc.ServiceContext) []service.Service {
   return []service.Service{
      // 监听延迟队列
      deferMq.NewAsynqTask(ctx, svcContext),

      // 监听定时任务
   }
}
```

go-zero-looklook/app/order/cmd/mq/internal/listen/asynqMqs.go就是定义的kq (go-queue的kafka)

```go
// kq
// 消息队列
func KqMqs(c config.Config, ctx context.Context, svcContext *svc.ServiceContext) []service.Service {
    return []service.Service{
        // 监听消费流水状态变更
        kq.MustNewQueue(c.PaymentUpdateStatusConf, kqMq.NewPaymentUpdateStatusMq(ctx, svcContext)),
        // .....
    }
}
```

### 2.3 实际业务

编写实际业务，我们就在go-zero-looklook/app/order/cmd/mq/internal/listen/mqs下，这里为了方便维护，也是做了分类

- deferMq : 延迟队列
- kq：消息队列

#### 2.3.1 延迟队列

```go
// 监听关闭订单
type AsynqTask struct {
   ctx    context.Context
   svcCtx *svc.ServiceContext
}

func NewAsynqTask(ctx context.Context, svcCtx *svc.ServiceContext) *AsynqTask {
   return &AsynqTask{
      ctx:    ctx,
      svcCtx: svcCtx,
   }
}

func (l *AsynqTask) Start() {
   fmt.Println("AsynqTask start ")

   srv := asynq.NewServer(
      asynq.RedisClientOpt{Addr: l.svcCtx.Config.Redis.Host, Password: l.svcCtx.Config.Redis.Pass},
      asynq.Config{
         Concurrency: 10,
         Queues: map[string]int{
            "critical": 6,
            "default":  3,
            "low":      1,
         },
      },
   )

   mux := asynq.NewServeMux()

   // 关闭民宿订单任务
   mux.HandleFunc(asynqmq.TypeHomestayOrderCloseDelivery, l.closeHomestayOrderStateMqHandler)

   if err := srv.Run(mux); err != nil {
      log.Fatalf("could not run server: %v", err)
   }
}

func (l *AsynqTask) Stop() {
   fmt.Println("AsynqTask stop")
}
```

因为 asynq 要先启动，然后定义路由任务，所以我们在asynqTask.go中做了统一的路由管理，之后我们每个业务都单独的在deferMq的文件夹下面定义一个文件（如“延迟关闭订单：closeHomestayOrderState.go”),这样每个业务一个文件，跟go-zero的api、rpc的logic一样，维护很方便

closeHomestayOrderState.go 关闭订单逻辑

```go
package deferMq

import (
    "context"
    "encoding/json"
    "looklook/app/order/cmd/rpc/order"
    "looklook/app/order/model"
    "looklook/common/asynqmq"
    "looklook/common/xerr"

    "github.com/hibiken/asynq"
    "github.com/pkg/errors"
)

func (l *AsynqTask) closeHomestayOrderStateMqHandler(ctx context.Context, t *asynq.Task) error {
    var p asynqmq.HomestayOrderCloseTaskPayload
    if err := json.Unmarshal(t.Payload(), &p); err != nil {
        return errors.Wrapf(xerr.NewErrMsg("解析asynq task payload err"), "closeHomestayOrderStateMqHandler payload err:%v, payLoad:%+v", err, t.Payload())
    }

    resp, err := l.svcCtx.OrderRpc.HomestayOrderDetail(ctx, &order.HomestayOrderDetailReq{
        Sn: p.Sn,
    })
    if err != nil || resp.HomestayOrder == nil {
        return errors.Wrapf(xerr.NewErrMsg("获取订单失败"), "closeHomestayOrderStateMqHandler 获取订单失败 or 订单不存在 err:%v, sn:%s ,HomestayOrder : %+v", err, p.Sn, resp.HomestayOrder)
    }

    if resp.HomestayOrder.TradeState == model.HomestayOrderTradeStateWaitPay {
        _, err := l.svcCtx.OrderRpc.UpdateHomestayOrderTradeState(ctx, &order.UpdateHomestayOrderTradeStateReq{
            Sn:         p.Sn,
            TradeState: model.HomestayOrderTradeStateCancel,
        })
        if err != nil {
            return errors.Wrapf(xerr.NewErrMsg("关闭订单失败"), "closeHomestayOrderStateMqHandler 关闭订单失败  err:%v, sn:%s ", err, p.Sn)
        }
    }

    return nil
}
```

#### 2.3.2 kq消息队列

看go-zero-looklook/app/order/cmd/mq/internal/mqs/kq文件夹下，因为kq跟asynq不太一样，它本身就是使用go-zero的Service管理的，已经实现了starter、stopper接口了，所以我们在/Users/seven/Developer/goenv/go-zero-looklook/app/order/cmd/mq/internal/listen/kqMqs.go中直接定义好一个go-queue业务扔给serviceGroup，去交给main启动就好了 , 我们的业务代码只需要实现go-queue的Consumer直接写我们自己业务即可。

1）/Users/seven/Developer/goenv/go-zero-looklook/app/order/cmd/mq/internal/listen/kqMqs.go

```go
func KqMqs(c config.Config, ctx context.Context, svcContext *svc.ServiceContext) []service.Service {
    return []service.Service{
        // 监听消费流水状态变更
        kq.MustNewQueue(c.PaymentUpdateStatusConf, kqMq.NewPaymentUpdateStatusMq(ctx, svcContext)),
        // .....
    }
}
```

可以看到kq.MustNewQueue本身返回就是 queue.MessageQueue ， queue.MessageQueue又实现了Start、Stop

2）业务中

/Users/seven/Developer/goenv/go-zero-looklook/app/order/cmd/mq/internal/mqs/kq/paymentUpdateStatus.go

```go
func (l *PaymentUpdateStatusMq) Consume(_, val string) error {
    fmt.Printf(" PaymentUpdateStatusMq Consume val : %s \n", val)
    // 解析数据
    var message kqueue.ThirdPaymentUpdatePayStatusNotifyMessage
    if err := json.Unmarshal([]byte(val), &message); err != nil {
        logx.WithContext(l.ctx).Error("PaymentUpdateStatusMq->Consume Unmarshal err : %v , val : %s", err, val)
        return err
    }

    // 执行业务..
    if err := l.execService(message); err != nil {
        logx.WithContext(l.ctx).Error("PaymentUpdateStatusMq->execService  err : %v , val : %s , message:%+v", err, val, message)
        return err
    }

    return nil
}
```

我们在paymentUpdateStatus.go中只需要实现接口Consume 就可以接受来自kq传过来的kafka的消息了，我们只管在我们Consumer中处理我们业务即可

## 3、定时任务

关于定时任务，目前go-zero-looklook没有使用，这里我也说明一下

- 如果你想简单一点直接使用cron（裸机、k8s都有），
- 如果稍微复杂一点可以使用[https://github.com/robfig/cron](https://link.segmentfault.com/?enc=wM%2Bzl6T68ucArh9y2Tw%2F6Q%3D%3D.8NijGlVmkkpmIY1tks3nJBFtLJACx2wsyz67sbikIiA%3D)包，在代码中定义时间
- 使用 xxl-job、gocron 分布式定时任务系统接入
- asynq 的 shedule

这里因为项目用的asynq，我就演示一下asynq的shedule吧

分为client与server ， client用来定义调度时间，server是到了时间接受client的消息触发来执行我们写的业务的，实际业务我们应该写在server，client用来定义业务调度时间的

asynqtest/docker-compose.yml

```yaml
version: '3'

services:
  #asynqmon asynq延迟队列、定时队列的webui
  asynqmon:
    image: hibiken/asynqmon:latest
    container_name: asynqmon_asynq
    ports:
      - 8980:8080
    command:
      - '--redis-addr=redis:6379'
      - '--redis-password=G62m50oigInC30sf'
    restart: always
    networks:
      - asynqtest_net
    depends_on:
      - redis
  
  #redis容器
  redis:
    image: redis:6.2.5
    container_name: redis_asynq
    ports:
      - 63779:6379
    environment:
      # 时区上海
      TZ: Asia/Shanghai
    volumes:
      # 数据文件
      - ./data/redis/data:/data:rw
    command: "redis-server --requirepass G62m50oigInC30sf  --appendonly yes"
    privileged: true
    restart: always
    networks:
      - asynqtest_net

networks:
  asynqtest_net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.22.0.0/16
```

asynqtest/shedule/client/client.go

```go
package main

import (
    "asynqtest/tpl"
    "encoding/json"
    "log"

    "github.com/hibiken/asynq"
)

const redisAddr = "127.0.0.1:63779"
const redisPwd = "G62m50oigInC30sf"

func main() {
    // 周期性任务
    scheduler := asynq.NewScheduler(
        asynq.RedisClientOpt{
            Addr:     redisAddr,
            Password: redisPwd,
        }, nil)

    payload, err := json.Marshal(tpl.EmailPayload{Email: "546630576@qq.com", Content: "发邮件呀"})
    if err != nil {
        log.Fatal(err)
    }

    task := asynq.NewTask(tpl.EMAIL_TPL, payload)
    // 每隔1分钟同步一次
    entryID, err := scheduler.Register("*/1 * * * *", task)

    if err != nil {
        log.Fatal(err)
    }
    log.Printf("registered an entry: %q\n", entryID)

    if err := scheduler.Run(); err != nil {
        log.Fatal(err)
    }
}
```

asynqtest/shedule/server/server.go

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "log"

    "asynqtest/tpl"

    "github.com/hibiken/asynq"
)

func main() {
    srv := asynq.NewServer(
        asynq.RedisClientOpt{Addr: "127.0.0.1:63779", Password: "G62m50oigInC30sf"},
        asynq.Config{
            Concurrency: 10,
            Queues: map[string]int{
                "critical": 6,
                "default":  3,
                "low":      1,
            },
        },
    )

    mux := asynq.NewServeMux()

    // 关闭民宿订单任务
    mux.HandleFunc(tpl.EMAIL_TPL, emailMqHandler)

    if err := srv.Run(mux); err != nil {
        log.Fatalf("could not run server: %v", err)
    }
}

func emailMqHandler(ctx context.Context, t *asynq.Task) error {
    var p tpl.EmailPayload
    if err := json.Unmarshal(t.Payload(), &p); err != nil {
        return fmt.Errorf("emailMqHandler err:%+v", err)
    }

    fmt.Printf("p : %+v \n", p)

    return nil
}
```

asynqtest/tpl/tpl.go

```go
package tpl

const EMAIL_TPL = "schedule:email"

type EmailPayload struct {
    Email   string
    Content string
}
```

启动 `server.go`、`client.go`

浏览器输入[http://127.0.0.1:8980/schedulers](https://link.segmentfault.com/?enc=SNFhaJXM2%2F0NOKgoVwq67A%3D%3D.JGAnHmIl5ZicfHzqVS89SUu3S2HCfiW5CNAKis1ruS8Evc6sJbjm0zi6PtR%2FBGcD)这里 可以看到所有client定义的任务

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041765891.png)

浏览器输入[http://127.0.0.1:8990/](https://link.segmentfault.com/?enc=5uM9dwe%2BEvO9v9A%2Fs%2FTJ7A%3D%3D.80t0ZbfcCo8%2FgIoaRTKv6ubWoO%2BjovhoXFtpxFEo5fY%3D)这里可以看到我们的server消费请

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041765892.png)

控制台消费情况

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041765893.png)

说一下asynq的shedule在集成到项目中的思路，可以单独启动一个服务作为调度client定义系统的定时任务调度管理，将server定义在每个业务自己的mq的asynq一起即可。

## 4、结尾

在这一节中，我们学会使用了消息队列、延迟队列 ，kafka可以通过管理工具去查看，至于asynq查看webui在go-zero-looklook/docker-compose-env.yml中我们已经启动好了asynqmon，直接使用[http://127.0.0.1:8980](https://link.segmentfault.com/?enc=h9M7uKPwkqw7G8fA8hRXgQ%3D%3D.527BdRmcElgDAmkhYBS9MbqvqmBwqWUBTxu5eEe3sp0%3D) 即可查看

# [第九篇、事务精讲](https://segmentfault.com/a/1190000041771652)

## 关于分布式事务

因为本项目服务划分相对独立一些，所以目前没有使用到分布式事务，不过go-zero结合dtm使用分布式事务的最佳实践，我有整理demo，这里就介绍一下go-zero结合dtm的使用，项目地址go-zero结合dtm最佳实践仓库地址 ： [https://github.com/Mikaelemmm...](https://link.segmentfault.com/?enc=psQzXv932jUXmEtISieDEA%3D%3D.w6Y58yOXeVxn4ptZDiR7rhCP5SH1uTbX%2BbLa4SPy32448byHRATMWLZhY%2FUQKgtz)

【注】下面说的不是go-zero-looklook项目，是这个项目 [https://github.com/Mikaelemmm...](https://link.segmentfault.com/?enc=W2O0WNjKijLR42iVWiMJKA%3D%3D.nigna8WgQ61vN4fsuVaA80HguposN0EOHsjavp3PW3pHOBB7kqdHPsZ2NaSAa0NF)

## 一、注意事项

- go-zero 1.2.4版本以上，这个一定要注意
- dtm 你用最新的就行了

## 二、clone dtm

```shell
git clone https://github.com/yedf/dtm.git
```

## 三、配置文件

1、找到项目跟文件夹下的conf.sample.yml

2、cp conf.sample.yml conf.yml

3、使用etcd ， 把配置中下面这段注释打开 （如果没用etcd就更简单了 ，这个都省了，直接链接到dtm server地址就可以了）

```yaml
MicroService:
  Driver: 'dtm-driver-gozero' # name of the driver to handle register/discover
  Target: 'etcd://localhost:2379/dtmservice' # register dtm server to this url
  EndPoint: 'localhost:36790'
```

解释一下：

MicroService 这个不要动，这个代表要对把dtm注册到那个微服务服务集群里面去，使微服务集群内部服务可以通过grpc直接跟dtm交互

Driver ：'dtm-driver-gozero' ， 使用go-zero的注册服务发现驱动，支持go-zero

Target: 'etcd://localhost:2379/dtmservice' 将当前dtm的server直接注册到微服务所在的etcd集群中，如果go-zero作为微服务使用的话，就可以直接通过etcd拿到dtm的server grpc链接，直接就可以跟dtm server交互了

EndPoint: 'localhost:36790' ， 代表的是dtm的server的连接地址+端口 ， 集群中的微服务可以直接通过etcd获得此地址跟dtm交互了，

如果你自己去改了dtm源码grpc端口，记得这里要改下端口

## 四、启动dtm server

在dtm项目根目录下

```shell
go run app/main.go dev
```

## 五、使用go-zero的grpc对接dtm

这是一个快速下单扣商品库存的例子

### 1、order-api

order-api是http服务入口创建订单

```go
service order {
   @doc "创建订单"
   @handler create
   post /order/quickCreate (QuickCreateReq) returns (QuickCreateResp)
}
```

接下来看logic

```go
func (l *CreateLogic) Create(req types.QuickCreateReq,r *http.Request) (*types.QuickCreateResp, error) {
    orderRpcBusiServer, err := l.svcCtx.Config.OrderRpcConf.BuildTarget()
    if err != nil{
        return nil,fmt.Errorf("下单异常超时")
    }
    stockRpcBusiServer, err := l.svcCtx.Config.StockRpcConf.BuildTarget()
    if err != nil{
        return nil,fmt.Errorf("下单异常超时")
    }

    createOrderReq:= &order.CreateReq{UserId: req.UserId,GoodsId: req.GoodsId,Num: req.Num}
    deductReq:= &stock.DecuctReq{GoodsId: req.GoodsId,Num: req.Num}

    // 这里只举了saga例子，tcc等其他例子基本没啥区别具体可以看dtm官网

    gid := dtmgrpc.MustGenGid(dtmServer)
    saga := dtmgrpc.NewSagaGrpc(dtmServer, gid).
        Add(orderRpcBusiServer+"/pb.order/create", orderRpcBusiServer+"/pb.order/createRollback", createOrderReq).
        Add(stockRpcBusiServer+"/pb.stock/deduct", stockRpcBusiServer+"/pb.stock/deductRollback", deductReq)

    err = saga.Submit()
    dtmimp.FatalIfError(err)
    if err != nil{
        return nil,fmt.Errorf("submit data to  dtm-server err  : %+v \n",err)
    }

    return &types.QuickCreateResp{}, nil
}
```

进入到下单逻辑时，分别获取order订单、stock库存服务的rpc在etcd中的地址，使用BuildTarget()这个方法

然后创建order、stock对应的请求参数

请求dtm获取全局事务id ， 基于此全局事务id开启grpc的saga分布式事务 ，将创建订单、扣减库存请求放入事务中，这里使用grpc形式请求，每个业务要有一个正向请求、一个回滚请求、以及请求参数，当执行其中任何一个业务正向请求出错了会自动调用事务中所有业务回滚请求达到回滚效果。

### 2、order-srv

order-srv是订单的rpc服务，与dtm-gozero-order数据库中order表交互

```protobuf
// service
service order {
   rpc create(CreateReq)returns(CreateResp);
   rpc createRollback(CreateReq)returns(CreateResp);
}
```

#### 2.1 Create

当order-api提交事务默认请求的是create方法，我们看看logic

```go
func (l *CreateLogic) Create(in *pb.CreateReq) (*pb.CreateResp, error) {
   fmt.Printf("创建订单 in : %+v \n", in)

   // barrier防止空补偿、空悬挂等具体看dtm官网即可，别忘记加barrier表在当前库中，因为判断补偿与要执行的sql一起本地事务
   barrier, err := dtmgrpc.BarrierFromGrpc(l.ctx)
   db, err := sqlx.NewMysql(l.svcCtx.Config.DB.DataSource).RawDB()
   if err != nil {
      // !!!一般数据库不会错误不需要dtm回滚，就让他一直重试，这时候就不要返回codes.Aborted, dtmcli.ResultFailure 就可以了，具体自己把控!!!
      return nil, status.Error(codes.Internal, err.Error())
   }
   if err := barrier.CallWithDB(db, func(tx *sql.Tx) error {

      order := new(model.Order)
      order.GoodsId = in.GoodsId
      order.Num = in.Num
      order.UserId = in.UserId

      _, err = l.svcCtx.OrderModel.Insert(tx, order)
      if err != nil {
         return fmt.Errorf("创建订单失败 err : %v , order:%+v \n", err, order)
      }

      return nil
   }); err != nil {
      // !!!一般数据库不会错误不需要dtm回滚，就让他一直重试，这时候就不要返回codes.Aborted, dtmcli.ResultFailure 就可以了，具体自己把控!!!
      return nil, status.Error(codes.Internal, err.Error())
   }

   return &pb.CreateResp{}, nil
}
```

可以看到，一进入方法内部我们就使用了dtm的子事务屏障技术，至于为什么使用子事务屏障是因为可能会出现重复请求或者空请求造成的脏数据等，在这里dtm自动给我们做了幂等处理不需要我们自己在额外做了，同时保证他内部的幂等处理跟我们自己执行的事务要在一个事务中，所以要使用一个会话的db链接，这时候我们就要先获取

```go
db, err := sqlx.NewMysql(l.svcCtx.Config.DB.DataSource).RawDB()
```

然后基于此db连接dtm在内部通过sql执行做幂等处理，同时我们基于此db连接开启事务，这样就能保证dtm内部的子事务屏障在执行sql操作与我们自己业务执行的sql操作在一个事务中。

dtm在使用grpc调用我们业务的时候，我们的grpc服务在返回给dtm server错误时候，dtm会根据我们返回给它的grpc错误码判断是要执行回滚操作还是一直重试：

- codes.Internal ： dtm server不会调用回滚，会一直重试，每次重试dtm的数据库中都会加一次重试次数，自己可以监控这个重试次数报警，人工处理
- codes.Aborted : dtm server会调用所有回滚请求，执行回滚操作

如果dtm在调用grpc返回错误是nil时候，就认为调用成功了

#### 2.2 CreateRollback

当我们调用订单的创建订单或者库存扣减时候返回给dtm server 的codes.Aborted时候，dtm server会调用所有回滚操作，CreateRollback就是对应订单下单的回滚操作，代码如下

```go
func (l *CreateRollbackLogic) CreateRollback(in *pb.CreateReq) (*pb.CreateResp, error) {
    fmt.Printf("订单回滚  , in: %+v \n", in)

    order, err := l.svcCtx.OrderModel.FindLastOneByUserIdGoodsId(in.UserId, in.GoodsId)
    if err != nil && err != model.ErrNotFound {
        // !!!一般数据库不会错误不需要dtm回滚，就让他一直重试，这时候就不要返回codes.Aborted, dtmcli.ResultFailure 就可以了，具体自己把控!!!
        return nil, status.Error(codes.Internal, err.Error())
    }

    if order != nil {

        barrier, err := dtmgrpc.BarrierFromGrpc(l.ctx)
        db, err := l.svcCtx.OrderModel.SqlDB()
        if err != nil {
            // !!!一般数据库不会错误不需要dtm回滚，就让他一直重试，这时候就不要返回codes.Aborted, dtmcli.ResultFailure 就可以了，具体自己把控!!!
            return nil, status.Error(codes.Internal, err.Error())
        }
        if err := barrier.CallWithDB(db, func(tx *sql.Tx) error {

            order.RowState = -1
            if err := l.svcCtx.OrderModel.Update(tx, order); err != nil {
                return fmt.Errorf("回滚订单失败  err : %v , userId:%d , goodsId:%d", err, in.UserId, in.GoodsId)
            }

            return nil
        }); err != nil {
            logx.Errorf("err : %v \n", err)

            // !!!一般数据库不会错误不需要dtm回滚，就让他一直重试，这时候就不要返回codes.Aborted, dtmcli.ResultFailure 就可以了，具体自己把控!!!
            return nil, status.Error(codes.Internal, err.Error())
        }

    }
    return &pb.CreateResp{}, nil
}
```

其实就是如果之前下单成功了，将之前下成功的单给取消掉就是对应下单的回滚操作

### 3、stock-srv

#### 3.1 Deduct

扣减库存，这里跟order的Create一样了，是下单事务内的正向操作，扣减库存，代码如下

```go
func (l *DeductLogic) Deduct(in *pb.DecuctReq) (*pb.DeductResp, error) {

    fmt.Printf("扣库存start....")

    stock, err := l.svcCtx.StockModel.FindOneByGoodsId(in.GoodsId)
    if err != nil && err != model.ErrNotFound {
        // !!!一般数据库不会错误不需要dtm回滚，就让他一直重试，这时候就不要返回codes.Aborted, dtmcli.ResultFailure 就可以了，具体自己把控!!!
        return nil, status.Error(codes.Internal, err.Error())
    }
    if stock == nil || stock.Num < in.Num {
        // 【回滚】库存不足确定需要dtm直接回滚，直接返回 codes.Aborted, dtmcli.ResultFailure 才可以回滚
        return nil, status.Error(codes.Aborted, dtmcli.ResultFailure)
    }

    // barrier防止空补偿、空悬挂等具体看dtm官网即可，别忘记加barrier表在当前库中，因为判断补偿与要执行的sql一起本地事务
    barrier, err := dtmgrpc.BarrierFromGrpc(l.ctx)
    db, err := l.svcCtx.StockModel.SqlDB()
    if err != nil {
        // !!!一般数据库不会错误不需要dtm回滚，就让他一直重试，这时候就不要返回codes.Aborted, dtmcli.ResultFailure 就可以了，具体自己把控!!!
        return nil, status.Error(codes.Internal, err.Error())
    }
    if err := barrier.CallWithDB(db, func(tx *sql.Tx) error {
        sqlResult,err := l.svcCtx.StockModel.DecuctStock(tx, in.GoodsId, in.Num)
        if err != nil{
            // !!!一般数据库不会错误不需要dtm回滚，就让他一直重试，这时候就不要返回codes.Aborted, dtmcli.ResultFailure 就可以了，具体自己把控!!!
            return status.Error(codes.Internal, err.Error())
        }
        affected, err := sqlResult.RowsAffected()
        if err != nil{
            // !!!一般数据库不会错误不需要dtm回滚，就让他一直重试，这时候就不要返回codes.Aborted, dtmcli.ResultFailure 就可以了，具体自己把控!!!
            return status.Error(codes.Internal, err.Error())
        }

        // 如果是影响行数为0，直接就告诉dtm失败不需要重试了
        if affected <= 0 {
            return  status.Error(codes.Aborted,  dtmcli.ResultFailure)
        }

        // ！！开启测试！！ 测试订单回滚更改状态为失效，并且当前库扣失败不需要回滚
        //return fmt.Errorf("扣库存失败 err : %v , in:%+v \n",err,in)

        return nil
    }); err != nil {
        // !!!一般数据库不会错误不需要dtm回滚，就让他一直重试，这时候就不要返回codes.Aborted, dtmcli.ResultFailure 就可以了，具体自己把控!!!
        return nil,err
    }

    return &pb.DeductResp{}, nil
}
```

这里值得注意的是当只有库存不足、或者在扣库存影响行数为0（未成功）才需要告诉dtm server要回滚，其他情况下其实都是网络抖动、硬件异常导致，应该让dtm server一直重试，当然自己要加个最大重试次数的监控报警，如果达到最大次数还未成功能实现自动发短信、打电话人工介入了。

#### 3.2 DeductRollback

这里是对应扣库存的回滚操作

```go
func (l *DeductRollbackLogic) DeductRollback(in *pb.DecuctReq) (*pb.DeductResp, error) {
    fmt.Printf("库存回滚 in : %+v \n", in)

    barrier, err := dtmgrpc.BarrierFromGrpc(l.ctx)
    db, err := l.svcCtx.StockModel.SqlDB()
    if err != nil {
        // !!!一般数据库不会错误不需要dtm回滚，就让他一直重试，这时候就不要返回codes.Aborted, dtmcli.ResultFailure 就可以了，具体自己把控!!!
        return nil, status.Error(codes.Internal, err.Error())
    }
    if err := barrier.CallWithDB(db, func(tx *sql.Tx) error {
        if err := l.svcCtx.StockModel.AddStock(tx, in.GoodsId, in.Num); err != nil {
            return fmt.Errorf("回滚库存失败 err : %v ,goodsId:%d , num :%d", err, in.GoodsId, in.Num)
        }
        return nil
    }); err != nil {
        logx.Errorf("err : %v \n", err)
        // !!!一般数据库不会错误不需要dtm回滚，就让他一直重试，这时候就不要返回codes.Aborted, dtmcli.ResultFailure 就可以了，具体自己把控!!!
        return nil, status.Error(codes.Internal, err.Error())
    }

    return &pb.DeductResp{}, nil
}
```

## 六、子事务屏障

这个词是dtm作者定义的，其实子事务屏障代码不多，看barrier.CallWithDB这个方法即可。

```go
// CallWithDB the same as Call, but with *sql.DB
func (bb *BranchBarrier) CallWithDB(db *sql.DB, busiCall BarrierBusiFunc) error {   
  tx, err := db.Begin()   
  if err != nil {      
    return err   
  }   
  return bb.Call(tx, busiCall)
}
```

由于这个方法他在内部开启本地事务，它内部是在此事务执行了sql操作，所以在我们执行自己的业务时候必须跟它用同一个事务，那就要基于同一个db连接开事务了，so~ 你知道为什么我们要提前获取db连接了吧，目的就是要让它内部执行的sql操作跟我们的sql操作在一个事务下。至于它内部为什么执行自己的sql操作，接下来我们分析。

我们看bb.Call这个方法

```go
// Call 子事务屏障，详细介绍见 https://zhuanlan.zhihu.com/p/388444465
// tx: 本地数据库的事务对象，允许子事务屏障进行事务操作
// busiCall: 业务函数，仅在必要时被调用
func (bb *BranchBarrier) Call(tx *sql.Tx, busiCall BarrierBusiFunc) (rerr error) {
 bb.BarrierID = bb.BarrierID + 1
 bid := fmt.Sprintf("%02d", bb.BarrierID)
 defer func() {
  // Logf("barrier call error is %v", rerr)
  if x := recover(); x != nil {
   tx.Rollback()
   panic(x)
  } else if rerr != nil {
   tx.Rollback()
  } else {
   tx.Commit()
  }
 }()
 ti := bb
 originType := map[string]string{
  BranchCancel:     BranchTry,
  BranchCompensate: BranchAction,
 }[ti.Op]

 originAffected, _ := insertBarrier(tx, ti.TransType, ti.Gid, ti.BranchID, originType, bid, ti.Op)
 currentAffected, rerr := insertBarrier(tx, ti.TransType, ti.Gid, ti.BranchID, ti.Op, bid, ti.Op)
 dtmimp.Logf("originAffected: %d currentAffected: %d", originAffected, currentAffected)
 if (ti.Op == BranchCancel || ti.Op == BranchCompensate) && originAffected > 0 || // 这个是空补偿
  currentAffected == 0 { // 这个是重复请求或者悬挂
  return
 }
 rerr = busiCall(tx)
 return
}
```

核心其实就是如下几行代码

```go
originAffected, _ := insertBarrier(tx, ti.TransType, ti.Gid, ti.BranchID, originType, bid, ti.Op)    
currentAffected, rerr := insertBarrier(tx, ti.TransType, ti.Gid, ti.BranchID, ti.Op, bid, ti.Op)    
dtmimp.Logf("originAffected: %d currentAffected: %d", originAffected, currentAffected)    
if (ti.Op == BranchCancel || ti.Op == BranchCompensate) && originAffected > 0 || // 这个是空补偿        
  currentAffected == 0 { // 这个是重复请求或者悬挂        
  return    
}
rerr = busiCall(tx)
func insertBarrier(tx DB, transType string, gid string, branchID string, op string, barrierID string, reason string) (int64, error) {
  if op == "" {        
    return 0, nil    
    }    

  sql := dtmimp.GetDBSpecial().GetInsertIgnoreTemplate("dtm_barrier.barrier(trans_type, gid, branch_id, op, barrier_id, reason) values(?,?,?,?,?,?)", "uniq_barrier")    

  return dtmimp.DBExec(tx, sql, transType, gid, branchID, op, barrierID, reason)
}
```

每一个业务逻辑，dtm server在正常成功请求时候， ti.Op 默认正常执行的操作是action，所以正常第一次请求都是ti.Op值都是action，那originType就是“”

```go
    originAffected, _ := insertBarrier(tx, ti.TransType, ti.Gid, ti.BranchID, originType, bid, ti.Op)
```

那么上面这个sql就不会执行因为ti.Op == "" 在insertBarrier中直接return了

```go
    currentAffected, rerr := insertBarrier(tx, ti.TransType, ti.Gid, ti.BranchID, ti.Op, bid, ti.Op)
```

那第二个sql的ti.Op是 action， 所以子事务屏障表barrier就会插入一条数据

同理在执行库存也会插入一条

### 1、整个事务都成功的子事务屏障

那在一次下订单正常成功请求下，由于 ti.Op都是action，所以originType都是"" , 所以不管是下单的barrier 还是扣库存的barrier，在执行他们2次barrier insert时候，originAffected都会忽略，因为originType==“” 会直接被return不插入数据，这样看来 不管是下单还是扣库存，barrier的第二条插入数据生效，所以barrier数据表中就会有2条下单数据，一条是订单的一条是扣库存的

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041771654.png)

gid : dtm全局事务id

branch_id : 每个全局事务id下的每个业务id

op : 操作，如果是正常成功请求就是action

barrier_id : 同一个业务下开多个会递增

这4个字段在表中是联合唯一索引，在insertBarrier时候，dtm判断如果存在就忽略不插入

#### 2、如果订单成功库存不足回滚子事务屏障

我们库存只有10个 ，我们下单20个

1）当订单下成功，因为订单下单时候并不知道后续库存情况（即使在下单时候先去查库存那也会有查询时候足够，扣除时候不足情况），

所以下单成功barrier表中按照之前梳理的逻辑就会在barrier表中产生一条正确数据执行数据

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041771655.png)

2）接着执行扣库存操作

```go
func (l *DeductLogic) Deduct(in *pb.DecuctReq) (*pb.DeductResp, error) {

    fmt.Printf("扣库存start....")

    stock, err := l.svcCtx.StockModel.FindOneByGoodsId(in.GoodsId)
    if err != nil && err != model.ErrNotFound {
        // !!!一般数据库不会错误不需要dtm回滚，就让他一直重试，这时候就不要返回codes.Aborted, dtmcli.ResultFailure 就可以了，具体自己把控!!!
        return nil, status.Error(codes.Internal, err.Error())
    }
    if stock == nil || stock.Num < in.Num {
        //【回滚】库存不足确定需要dtm直接回滚，直接返回 codes.Aborted, dtmcli.ResultFailure 才可以回滚
        return nil, status.Error(codes.Aborted, dtmcli.ResultFailure)
    }
  
  .......
}
```

在执行扣库存业务逻辑之前，由于我们查询库存发现库存不足，所以直接return codes.Aborted 了，不会走到子事务屏障barrier这里，所以barrier表中不会插入数据，而是告诉dtm要回滚

3）调用order回滚操作

订单回滚的时候会开启barrier，这时候又会执行barrier代码（如下），由于回滚代码的ti.Op是compensate ，orginType就是action

```go
originAffected, _ := insertBarrier(tx, ti.TransType, ti.Gid, ti.BranchID, originType, bid, ti.Op)
currentAffected, err := insertBarrier(tx, ti.TransType, ti.Gid, ti.BranchID, ti.Op, bid, ti.Op)

dtmimp.Logf("originAffected: %d currentAffected: %d", originAffected, currentAffected)

if (ti.Op == BranchCancel || ti.Op == BranchCompensate) && originAffected > 0 || // 这个是空补偿        
  currentAffected == 0 { // 这个是重复请求或者悬挂        
  return    
}
rerr = busiCall(tx)
```

由于我们之前下订单成功了，barrier表里有一条下单成功时候的记录action，所以originAffected==0 ，所以只会插入一条当前回滚记录继续调用 busiCall(tx) 执行后续我们自己写的回滚操作

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041771656.png)

到此，我们应该只有两条数据，一条订单成功创建记录，一条订单回滚记录

4）库存回滚DeductRollback

订单回滚成功后，会再继续调用库存回滚DeductRollback，库存回滚代码如下

这就是子事务屏障自动帮我们判断的，也就是那两条核心插入语句帮我们判断的，以至于我们业务不会出现脏数据

库存这里回滚分两种情况

- 没扣成功回滚
- 扣成功回滚

###### 没扣成功回滚（我们当前举例场景是这个 ）

首先调用库存回滚时候ti.Op是compensate ，orginType就是action ，会执行下面2条insert

```go
originAffected, _ := insertBarrier(tx, ti.TransType, ti.Gid, ti.BranchID, originType, bid, ti.Op)
currentAffected, err := insertBarrier(tx, ti.TransType, ti.Gid, ti.BranchID, ti.Op, bid, ti.Op)

dtmimp.Logf("originAffected: %d currentAffected: %d", originAffected, currentAffected)

if (ti.Op == BranchCancel || ti.Op == BranchCompensate) && originAffected > 0 || // 这个是空补偿
  currentAffected == 0 { // 这个是重复请求或者悬挂
  return}rerr = busiCall(tx)
}
```

这里结合判断如果是回滚、取消操作，originAffected > 0 当前插入成功了，之前对应的正向扣库存操作没有插入成功，说明之前库存没扣成功，直接return就不需要执行后续补偿了。所以此时会在barrier表中插入2条数据直接return，就不会执行我们后续补偿操作了

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041771657.png)

到此我们barrier表中有4条数据了

###### 扣成功回滚（这个情况自己可以尝试模拟此场景）

如果我们上一步扣库存成功，在执行此补偿的时候ti.Op是compensate ，orginType就是action ，继续执行2个insert语句

```go
originAffected, _ := insertBarrier(tx, ti.TransType, ti.Gid, ti.BranchID, originType, bid, ti.Op)
currentAffected, err := insertBarrier(tx, ti.TransType, ti.Gid, ti.BranchID, ti.Op, bid, ti.Op)

dtmimp.Logf("originAffected: %d currentAffected: %d", originAffected, currentAffected)

if (ti.Op == BranchCancel || ti.Op == BranchCompensate) && originAffected > 0 || // 这个是空补偿
  currentAffected == 0 { // 这个是重复请求或者悬挂
  return}rerr = busiCall(tx)
}
```

这里结合判断如果是回滚、取消操作，originAffected == 0 当前插入忽略了没插入进去，说明之前正向扣库存插入成功了，这里只插入第二个sql语句记录即可，然后在执行后续我们补偿的业务操作。

所以，整体分析下来核心语句就是2条insert，它帮我们解决了重复回滚数据、数据幂等情况，只能说dtm作者想法真的很好，用了最少的代码帮我们解决了一个很麻烦的问题

## 七、go-zero对接中注意事项

### 1、dtm的回滚补偿

在使用dtm的grpc时候，当我们使用saga、tcc等如果第一步尝试或者执行失败了，是希望它能执行后面的rollback的，在grpc中的服务如果发生错误了，必须返回 ： status.Error(codes.Aborted, dtmcli.ResultFailure) ， 返回其他错误，不会执行你的rollback操作，dtm会一直重试，如下：

```go
stock, err := l.svcCtx.StockModel.FindOneByGoodsId(in.GoodsId)
if err != nil && err != model.ErrNotFound {
  // !!!一般数据库不会错误不需要dtm回滚，就让他一直重试，这时候就不要返回codes.Aborted, dtmcli.ResultFailure 就可以了，具体自己把控!!!
  return nil, status.Error(codes.Internal, err.Error())
}
if stock == nil || stock.Num < in.Num {
  //【回滚】库存不足确定需要dtm直接回滚，直接返回 codes.Aborted, dtmcli.ResultFailure 才可以回滚
  return nil, status.Error(codes.Aborted, dtmcli.ResultFailure)
}
```

### 2、barrier的空补偿、悬挂等

之前准备工作中，我们创建了dtm_barrier库以及执行了barrier.mysql.sql，这个其实就是为我们的业务服务做了一个检查，防止空补偿，具体可以看barrier.Call中源码，没几行代码可以看懂的。

如果我们线上使用的话，你的每个与db交互的服务只要用到了barrier，这个服务使用到的mysql账号，要给他分配barrier库的权限，这个不要忘记了

### 3、barrier在rpc中本地事务

在rpc的业务中，如果使用了barrier的话，那么在model中与db交互时候必须要用事务，并且一定要跟barrier用同一个事务

logic

```go
barrier, err := dtmgrpc.BarrierFromGrpc(l.ctx)
    db, err := sqlx.NewMysql(l.svcCtx.Config.DB.DataSource).RawDB()
    if err != nil {
        // !!!一般数据库不会错误不需要dtm回滚，就让他一直重试，这时候就不要返回codes.Aborted, dtmcli.ResultFailure 就可以了，具体自己把控!!!
        return nil, status.Error(codes.Internal, err.Error())
    }
    if err := barrier.CallWithDB(db, func(tx *sql.Tx) error {
        sqlResult,err := l.svcCtx.StockModel.DecuctStock(tx, in.GoodsId, in.Num)
        if err != nil{
            // !!!一般数据库不会错误不需要dtm回滚，就让他一直重试，这时候就不要返回codes.Aborted, dtmcli.ResultFailure 就可以了，具体自己把控!!!
            return status.Error(codes.Internal, err.Error())
        }
        affected, err := sqlResult.RowsAffected()
        if err != nil{
            // !!!一般数据库不会错误不需要dtm回滚，就让他一直重试，这时候就不要返回codes.Aborted, dtmcli.ResultFailure 就可以了，具体自己把控!!!
            return status.Error(codes.Internal, err.Error())
        }

        // 如果是影响行数为0，直接就告诉dtm失败不需要重试了
        if affected <= 0 {
            return  status.Error(codes.Aborted,  dtmcli.ResultFailure)
        }

        // ！！开启测试！！ ： 测试订单回滚更改状态为失效，并且当前库扣失败不需要回滚
        // return fmt.Errorf("扣库存失败 err : %v , in:%+v \n",err,in)

        return nil
    }); err != nil {
        // !!!一般数据库不会错误不需要dtm回滚，就让他一直重试，这时候就不要返回codes.Aborted, dtmcli.ResultFailure 就可以了，具体自己把控!!!
        return nil, err
    }
```

model

```go
func (m *defaultStockModel) DecuctStock(tx *sql.Tx,goodsId , num int64) (sql.Result,error) {
    query := fmt.Sprintf("update %s set `num` = `num` - ? where `goods_id` = ? and num >= ?", m.table)
    return tx.Exec(query,num, goodsId,num)
}

func (m *defaultStockModel) AddStock(tx *sql.Tx,goodsId , num int64) error {
    query := fmt.Sprintf("update %s set `num` = `num` + ? where `goods_id` = ?", m.table)
    _, err :=tx.Exec(query, num, goodsId)
    return err
}
```

## 七、使用go-zero的http对接

这个基本没啥难度，grpc会了这个很简单，鉴于go在微服务中去使用http场景不多，这里就不详细去做了，我之前一个版本写过一个简单的，但是没这个完善，有兴趣可以去看下，不过那个barrier是自己基于go-zero的sqlx，将dtm的官方的做了修改，现在不需要了。

项目地址：[https://github.com/Mikaelemmmm/dtmbarrier-go-zero](https://link.segmentfault.com/?enc=H5bfriToX6ngfK3flbsL5A%3D%3D.zT1zMoMuUl08dBn54LoYnjULPlXG3n%2Bvro2dz%2FmXyOCFSfHgsAkBKCOVIdEQ0iR6xhYtNcKT9TicijNwsrlRHQ%3D%3D)

# [第十篇、错误处理](https://segmentfault.com/a/1190000041775563)

## 1、概述

我们在平时开发时候，程序在出错时，希望可以通过错误日志能快速定位问题（那么传递进来的参数、包括堆栈信息肯定就要都要打印到日志），但同时又想返回给前端用户比较友善、能看得懂的错误提示，那这两点如果只通过一个fmt.Error、errors.new等返回一个错误信息肯定是无法做到的，除非在返回前端错误提示的地方同时在记录log，这样的话日志满天飞，代码难看不说，日志到时候也会很难看。

那么我们想一下，如果有一个统一的地方记录日志，同时在业务代码中只需要一个return err 就能将返回给前端的错误提示信息、日志记录相信信息分开提示跟记录，如果按照这个思路实现，那简直不要太爽，是的 go-zero-looklook就是这么处理的，接下来我们看下。

## 2、rpc错误处理

按照正常情况下，go-zero的rpc服务是基于grpc的，默认返回的错误是grpc的status.Error 没法给我们自定义的错误合并，并且也不适合我们自定义的错误，它的错误码、错误类型都是定义死在grpc包中的，ok ，如果我们在rpc中能用自定义错误返回，然后在拦截器统一返回时候转成grpc的status.Error ， 那么我们rpc的err跟api的err是不是可以统一管理我们自己的错误了呢？

我们看一下grpc的status.Error的code里面是什么

```go
package codes // import "google.golang.org/grpc/codes"

import (
    "fmt"
    "strconv"
)

// A Code is an unsigned 32-bit error code as defined in the gRPC spec.
type Code uint32
.......
```

grpc的err对应的错误码其实就是一个uint32 ， 我们自己定义错误用uint32然后在rpc的全局拦截器返回时候转成grpc的err，就可以了

所以我们自己定义全局错误码在app/common/xerr

errCode.go

```go
package xerr

// 成功返回
const OK uint32 = 200

// 前3位代表业务,后三位代表具体功能

// 全局错误码
const SERVER_COMMON_ERROR uint32 = 100001
const REUQES_PARAM_ERROR uint32 = 100002
const TOKEN_EXPIRE_ERROR uint32 = 100003
const TOKEN_GENERATE_ERROR uint32 = 100004
const DB_ERROR uint32 = 100005

// 用户模块
```

errMsg.go

```go
package xerr

var message map[uint32]string

func init() {
   message = make(map[uint32]string)
   message[OK] = "SUCCESS"
   message[SERVER_COMMON_ERROR] = "服务器开小差啦,稍后再来试一试"
   message[REUQES_PARAM_ERROR] = "参数错误"
   message[TOKEN_EXPIRE_ERROR] = "token失效，请重新登陆"
   message[TOKEN_GENERATE_ERROR] = "生成token失败"
   message[DB_ERROR] = "数据库繁忙,请稍后再试"
}

func MapErrMsg(errcode uint32) string {
   if msg, ok := message[errcode]; ok {
      return msg
   } else {
      return "服务器开小差啦,稍后再来试一试"
   }
}

func IsCodeErr(errcode uint32) bool {
   if _, ok := message[errcode]; ok {
      return true
   } else {
      return false
   }
}
```

errors.go

```go
package xerr

import "fmt"

// 常用通用固定错误
type CodeError struct {
   errCode uint32
   errMsg  string
}

// 返回给前端的错误码
func (e *CodeError) GetErrCode() uint32 {
   return e.errCode
}

// 返回给前端显示端错误信息
func (e *CodeError) GetErrMsg() string {
   return e.errMsg
}

func (e *CodeError) Error() string {
   return fmt.Sprintf("ErrCode:%d，ErrMsg:%s", e.errCode, e.errMsg)
}

func NewErrCodeMsg(errCode uint32, errMsg string) *CodeError {
   return &CodeError{errCode: errCode, errMsg: errMsg}
}
func NewErrCode(errCode uint32) *CodeError {
   return &CodeError{errCode: errCode, errMsg: MapErrMsg(errCode)}
}

func NewErrMsg(errMsg string) *CodeError {
   return &CodeError{errCode: SERVER_COMMON_ERROR, errMsg: errMsg}
}
```

比如我们在用户注册时候的rpc代码

```go
package logic

import (
    "context"

    "looklook/app/identity/cmd/rpc/identity"
    "looklook/app/usercenter/cmd/rpc/internal/svc"
    "looklook/app/usercenter/cmd/rpc/usercenter"
    "looklook/app/usercenter/model"
    "looklook/common/xerr"

    "github.com/pkg/errors"
    "github.com/tal-tech/go-zero/core/logx"
    "github.com/tal-tech/go-zero/core/stores/sqlx"
)

var ErrUserAlreadyRegisterError = xerr.NewErrMsg("该用户已被注册")

type RegisterLogic struct {
    ctx    context.Context
    svcCtx *svc.ServiceContext
    logx.Logger
}

func NewRegisterLogic(ctx context.Context, svcCtx *svc.ServiceContext) *RegisterLogic {
    return &RegisterLogic{
        ctx:    ctx,
        svcCtx: svcCtx,
        Logger: logx.WithContext(ctx),
    }
}

func (l *RegisterLogic) Register(in *usercenter.RegisterReq) (*usercenter.RegisterResp, error) {

    user, err := l.svcCtx.UserModel.FindOneByMobile(in.Mobile)
    if err != nil && err != model.ErrNotFound {
        return nil, errors.Wrapf(xerr.ErrDBError, "mobile:%s,err:%v", in.Mobile, err)
    }

    if user != nil {
        return nil, errors.Wrapf(ErrUserAlreadyRegisterError, "用户已经存在 mobile:%s,err:%v", in.Mobile, err)
    }

    var userId int64

    if err := l.svcCtx.UserModel.Trans(func(session sqlx.Session) error {

        user := new(model.User)
        user.Mobile = in.Mobile
        user.Nickname = in.Nickname
        insertResult, err := l.svcCtx.UserModel.Insert(session, user)
        if err != nil {
            return errors.Wrapf(xerr.ErrDBError, "err:%v,user:%+v", err, user)
        }
        lastId, err := insertResult.LastInsertId()
        if err != nil {
            return errors.Wrapf(xerr.ErrDBError, "insertResult.LastInsertId err:%v,user:%+v", err, user)
        }
        userId = lastId

        userAuth := new(model.UserAuth)
        userAuth.UserId = lastId
        userAuth.AuthKey = in.AuthKey
        userAuth.AuthType = in.AuthType
        if _, err := l.svcCtx.UserAuthModel.Insert(session, userAuth); err != nil {
            return errors.Wrapf(xerr.ErrDBError, "err:%v,userAuth:%v", err, userAuth)
        }
        return nil
    }); err != nil {
        return nil, err
    }

    // 2、生成token.
    resp, err := l.svcCtx.IdentityRpc.GenerateToken(l.ctx, &identity.GenerateTokenReq{
        UserId: userId,
    })
    if err != nil {
        return nil, errors.Wrapf(ErrGenerateTokenError, "IdentityRpc.GenerateToken userId : %d , err:%+v", userId, err)
    }

    return &usercenter.RegisterResp{
        AccessToken:  resp.AccessToken,
        AccessExpire: resp.AccessExpire,
        RefreshAfter: resp.RefreshAfter,
    }, nil
}
errors.Wrapf(ErrUserAlreadyRegisterError, "用户已经存在 mobile:%s,err:%v", in.Mobile, err)
```

这里我们使用go默认的errors的包的errors.Wrapf （ 如果这里不明白就去查一下go的errors包下的Wrap、 Wrapf等）

第一个参数， ErrUserAlreadyRegisterError 定义在上方 就是使用xerr.NewErrMsg("该用户已被注册") ， 返回给前端友好的提示，要记住这里用的是我们xerr包下的方法

第二个参数，就是记录在服务器日志，可以写详细一点都没关系只会记录在服务器不会被返回给前端

那我们来看看为什么第一个参数就能是返回给前端的，第二个参数就是记录日志的

⚠️【注】我们在rpc的启动文件main方法中，加了grpc的全局拦截器，这个很重要 ，如果不加这个没办法实现

```go
package main

......

func main() {
    ........

    //rpc log,grpc的全局拦截器
    s.AddUnaryInterceptors(rpcserver.LoggerInterceptor)

    .......
}
```

我们看看rpcserver.LoggerInterceptor的具体实现

```go
import (
    ...
    "github.com/pkg/errors"
)

func LoggerInterceptor(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (resp interface{}, err error) {
   resp, err = handler(ctx, req)
   if err != nil {
      causeErr := errors.Cause(err)                // err类型
      if e, ok := causeErr.(*xerr.CodeError); ok { //自定义错误类型
         logx.WithContext(ctx).Errorf("【RPC-SRV-ERR】 %+v", err)

         //转成grpc err
         err = status.Error(codes.Code(e.GetErrCode()), e.GetErrMsg())
      } else {
         logx.WithContext(ctx).Errorf("【RPC-SRV-ERR】 %+v", err)
      }
   }

   return resp, err
}
```

当有请求进入到rpc服务时候，先进入拦截器然后就是执行handler方法，如果你想在进入之前处理某些事情就可以写在handler方法之前，那我们想处理的是返回结果如果有错误的情况，所以我们在handler下方使用了github.com/pkg/errors这个包，这个包处理错误是go中经常用到的这不是官方的errors包，但是设计的很好，go官方的Wrap、Wrapf等就是借鉴了这个包的思路。

因为我们grpc内部业务在返回错误时候

 1）如果是我们自己业务错误，我们会统一用xerr生成错误，这样就可以拿到我们定义的错误信息，因为前面我们自己错误也是用的uint32，所以在这里统一转成 grpc错误err = status.Error(codes.Code(e.GetErrCode()), e.GetErrMsg())，那这里获取到的，e.GetErrCode()就是我们定义的code，e.GetErrMsg() 就是我们之前定义返回的错误第二个参数

2）但是还有一种情况是rpc服务异常了底部抛出来的错误，本身就是grpc错误了，那这种的我们直接就记录异常就好了

## 3、api错误

当我们api在logic中调用rpc的Register时候，rpc返回了上面第2步的错误信息 代码如下

```go
......
func (l *RegisterLogic) Register(req types.RegisterReq) (*types.RegisterResp, error) {
    registerResp, err := l.svcCtx.UsercenterRpc.Register(l.ctx, &usercenter.RegisterReq{
        Mobile:   req.Mobile,
        Nickname: req.Nickname,
        AuthKey:  req.Mobile,
        AuthType: model.UserAuthTypeSystem,
    })
    if err != nil {
        return nil, errors.Wrapf(err, "req: %+v", req)
    }

    var resp types.RegisterResp
    _ = copier.Copy(&resp, registerResp)

    return &resp, nil
}
```

这里同样是使用标准包的errors.Wrapf ， 也就是说所有我们业务中返回错误都适用标准包的errors，但是内部参数要使用我们xerr定义的错误

这里有2个注意点

1）api服务想把rpc返回给前端友好的错误提示信息，我们想直接返回给前端不做任何处理（比如rpc已经返回了“用户已存在”，api不想做什么处理，就想把这个错误信息直接返回给前端）

针对这种情况，直接就像上图这种写就可以了，将rpc调用处的err直接作为errors.Wrapf 第一个参数扔出去，但是第二个参数最好记录一下自己需要的详细日志方便后续在api log里查看

2）api服务不管rpc返回的是什么错误信息，我就想自己再重新定义给前端返回错误信息（比如rpc已经返回了“用户已存在”，api想调用rpc时只要有错误我就返回给前端“用户注册失败”）

针对这种情况，如下这样写即可（当然你可以将xerr.NewErrMsg("用户注册失败") 放到代码上方使用一个变量，这里放变量也可以）

```go
func (l *RegisterLogic) Register(req types.RegisterReq) (*types.RegisterResp, error) {
    .......
    if err != nil {
        return nil, errors.Wrapf(xerr.NewErrMsg("用户注册失败"), "req: %+v,rpc err:%+v", req,err)
    }
    .....
}
```

接下来我们看最终返回给前端怎么处理的，我们接着看app/usercenter/cmd/api/internal/handler/user/registerHandler.go

```go
func RegisterHandler(ctx *svc.ServiceContext) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        var req types.RegisterReq
        if err := httpx.Parse(r, &req); err != nil {
            httpx.Error(w, err)
            return
        }

        l := user.NewRegisterLogic(r.Context(), ctx)
        resp, err := l.Register(req)
        result.HttpResult(r, w, resp, err)
    }
}
```

这里可以看到，go-zero-looklook生成的handler代码 有2个地方跟默认官方的goctl生成的代码不一样，就是在处理错误处理的时候，这里替换成我们自己的错误处理了，在common/result/httpResult.go

【注】有人会说，每次使用goctl都要过来手动改，那不是要麻烦死了，这里我们使用go-zero给我们提供的template模版功能（还不知道这个的就要去官方文档学习一下了），修改一下handler生成模版即可，整个项目的模版文件放在deploy/goctl下，这里hanlder修改的模版在deploy/goctl/1.2.3-cli/api/handler.tpl

ParamErrorResult很简单，专门处理参数错误的

```stylus
// http 参数错误返回
func ParamErrorResult(r *http.Request, w http.ResponseWriter, err error) {
   errMsg := fmt.Sprintf("%s ,%s", xerr.MapErrMsg(xerr.REUQES_PARAM_ERROR), err.Error())
   httpx.WriteJson(w, http.StatusBadRequest, Error(xerr.REUQES_PARAM_ERROR, errMsg))
}
```

我们主要来看HttpResult ， 业务返回的错误处理的

```go
// http返回
func HttpResult(r *http.Request, w http.ResponseWriter, resp interface{}, err error) {
    if err == nil {
        // 成功返回
        r := Success(resp)
        httpx.WriteJson(w, http.StatusOK, r)
    } else {
        // 错误返回
        errcode := xerr.SERVER_COMMON_ERROR
        errmsg := "服务器开小差啦，稍后再来试一试"

        causeErr := errors.Cause(err) // err类型
        if e, ok := causeErr.(*xerr.CodeError); ok {
            // 自定义错误类型
            // 自定义CodeError
            errcode = e.GetErrCode()
            errmsg = e.GetErrMsg()
        } else {
            if gstatus, ok := status.FromError(causeErr); ok {
                // grpc err错误
                grpcCode := uint32(gstatus.Code())
                if xerr.IsCodeErr(grpcCode) {
                    // 区分自定义错误跟系统底层、db等错误，底层、db错误不能返回给前端
                    errcode = grpcCode
                    errmsg = gstatus.Message()
                }
            }
        }

        logx.WithContext(r.Context()).Errorf("【API-ERR】 : %+v ", err)
        httpx.WriteJson(w, http.StatusBadRequest, Error(errcode, errmsg))
    }
}
```

err ： 要记录的日志错误

errcode ： 返回给前端的错误码

errmsg ：返回给前端的友好的错误提示信息

成功直接返回，如果遇到错误了，也是使用github.com/pkg/errors这个包来判断错误，是不是我们自己定义的错误（api中定义的错误直接使用我们自己定义的xerr），还是grpc错误（rpc业务抛出来的），如果是grpc错误在通过uint32转成我们自己错误码，根据错误码再去我们自己定义错误信息中找到定义的错误信息返回给前端，如果是api错误直接返回给前端我们自己定义的错误信息，都找不到那就返回默认错误“服务器开小差了” ,

## 4、结尾

到这里错误处理已经消息描述清楚了，接下来我们要看打印了服务端的错误日志，我们该如何收集查看，就涉及到日志收集系统。

# [第十一篇、日志收集](https://segmentfault.com/a/1190000041791801)

## 序言

在介绍之前，我先说一下整体思路，如果你的业务日志量不是特别大恰好你又使用的是云服务，那你直接使用云服务日志就可以了，比如阿里云的SLS，基本就是点点鼠标配置几步就可以将你的日志收集到阿里云的SLS里面了，直接就可以在阿里云中查看收集上来的日志了，感觉也没必要折腾。

如果你的日志量比较大，那就可以上日志系统了。

## 1、日志系统

我们将业务日志打印到console、file之后，市面上比较常用的方式是elk、efk等基本思路一样，我们拿常说的elk来举例，基本思路就是logstash收集过滤到elasticsearch中，然后kibana呈现

但是logstash本身是使用java开发的，占用资源是真滴高，我们用go做业务，本身除了快就是占用资源少构建块，现在在搞个logstash浪费资源，那我们使用go-stash替代logstash，go-stash是go-zero官方自己开发的并且在线上经过长期大量实践的，但是它不负责收集日志，只负责过滤收集上来信息。

go-stash: [https://github.com/kevwan/go-...](https://link.segmentfault.com/?enc=1qofRuPT0%2FN0nqyM2%2BxR5w%3D%3D.5DRiDl2IrV1UWl5AZlmXEiiO4Nk9gk7xKyt2eI0SlMh6d9cnJDWOtEYSRN4UI007)

## 2、架构方案

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041791803.png)

filebeat收集我们的业务日志，然后将日志输出到kafka中作为缓冲，go-stash获取kafka中日志根据配置过滤字段，然后将过滤后的字段输出到elasticsearch中，最后由kibana负责呈现日志

## 3、实现方案

在上一节错误处理中，我们可以看到已经将我们想要的错误日志打印到了console控制台中了，现在我们只需要做后续收集即可

### 3.1 kafka

```yaml
#消息队列
kafka:
  image: wurstmeister/kafka
  container_name: kafka
  ports:
    - 9092:9092
  environment:
    KAFKA_ADVERTISED_HOST_NAME: kafka
    KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    TZ: Asia/Shanghai
  restart: always
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock
  networks:
    - looklook_net
  depends_on:
    - zookeeper
```

先配置好kafka、zookeeper

然后我们进入kafka中先创建好filebeat收集日志到kafka的topic

进入kafka容器

```shell
$  docker exec -it kafka /bin/sh
```

修改kafka监听配置(或者你把配置文件挂载到物理机在修改也可以)

```bash
$ vi /opt/kafka/config/server.properties
listeners=PLAINTEXT://kafka:9092 # 原文件中，要加kafka listeners=PLAINTEXT://:9092
advertised.listeners=PLAINTEXT://kafka:9092 #源文件中，要加kafka advertised.listeners=PLAINTEXT://:9092
```

创建topic

```shell
$  cd /opt/kafka/bin
$ ./kafka-topics.sh --create --zookeeper zookeeper:2181 --replication-factor 1 -partitions 1 --topic looklook-log
```

### 3.2 filebeat

在项目根目录下 docker-compose-env.yml文件中可以看到我们配置了filebeat

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041791804.png)

filebeat的配置我们挂载到 deploy/filebeat/conf/filebeat.yml

```yaml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/lib/docker/containers/*/*-json.log

filebeat.config:
  modules:
    path: ${path.config}/modules.d/*.yml
    reload.enabled: false

processors:
  - add_cloud_metadata: ~
  - add_docker_metadata: ~

output.kafka:
  enabled: true
  hosts: ["kafka:9092"]
  #要提前创建topic
  topic: "looklook-log"
  partition.hash:
    reachable_only: true
  compression: gzip
  max_message_bytes: 1000000
  required_acks: 1
```

配置比较简单，可以看到我们收集所有日志直接 输出到我们配置的kafka中 , topic配置上一步kafka中创建的topic即可

### 3.3 配置go-stash

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041791805.png)

我们来看下go-stash的配置文件 deploy/go-stash/etc/config.yaml

```yaml
Clusters:
  - Input:
      Kafka:
        Name: gostash
        Brokers:
          - "kafka:9092"
        Topics:
          - looklook-log
        Group: pro
        Consumers: 16
    Filters:
      - Action: drop
        Conditions:
          - Key: k8s_container_name
            Value: "-rpc"
            Type: contains
          - Key: level
            Value: info
            Type: match
            Op: and
      - Action: remove_field
        Fields:
          # - message
          - _source
          - _type
          - _score
          - _id
          - "@version"
          - topic
          - index
          - beat
          - docker_container
          - offset
          - prospector
          - source
          - stream
          - "@metadata"
      - Action: transfer
        Field: message
        Target: data
    Output:
      ElasticSearch:
        Hosts:
          - "http://elasticsearch:9200"
        Index: "looklook-{{yyyy-MM-dd}}"
```

配置消费的kafka以及输出的elasticsearch ， 以及要过滤的字段等

### 3.4 elastic search、kibana

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041791806.png)

访问kibana [http://127.0.0.1:5601/](https://link.segmentfault.com/?enc=7piipqyhL1w1uTIXAs8Wsg%3D%3D.6gvuSbT8%2FWhriCptQ%2BnC2kbybtWQhVQ%2BczsNlsTVrvU%3D) ， 创建日志索引

点击左上角菜单(三个横线那个东东)，找到Analytics - > 点击discover

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041791807.png)

然后在当前页面，Create index pattern->输入looklook-* -> Next Step ->选择@timestamp->Create index pattern

然后点击左上角菜单，找到Analytics->点击discover ，稍等一会，日志都显示了 （如果不显示，就去排查filebeat、go-stash，使用docker logs -f filebeat查看）

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041791808.png)

我们在代码中添加一个错误日志尝试一下，代码如下

```go
func (l *BusinessListLogic) BusinessList(req types.BusinessListReq) (*types.BusinessListResp, error) {

    logx.Error("测试的日志")

    ...
}
```

我们访问这个业务方法，去kibana中搜索 data.log : "测试"，如下图

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041791809.png)

#### 4、结尾

到此日志收集就完成了，接下来我们要实现链路追踪

# [第十二篇、链路追踪](https://segmentfault.com/a/1190000041798112)

## 1、概述

如果按照我前两节错误处理、日志收集配置的话，我们通过日志中的traceId也可以完整看到报错时候的整体链路日志，但是不报错的时候或者想方便的查看单个业务整个链路调用的执行时间是不太方便查看的，所以最好还是加上链路追踪。

go-zero底层已经帮我们把代码跟链路追踪对接的代码已经写好了

```go
func startAgent(c Config) error {
  opts := []sdktrace.TracerProviderOption{
    // Set the sampling rate based on the parent span to 100%
    sdktrace.WithSampler(sdktrace.ParentBased(sdktrace.TraceIDRatioBased(c.Sampler))),
    // Record information about this application in an Resource.
    sdktrace.WithResource(resource.NewSchemaless(semconv.ServiceNameKey.String(c.Name))),
  }

  if len(c.Endpoint) > 0 {
    exp, err := createExporter(c)
    if err != nil {
      logx.Error(err)
      return err
    }

    // Always be sure to batch in production.
    opts = append(opts, sdktrace.WithBatcher(exp))
  }

  tp := sdktrace.NewTracerProvider(opts...)
  otel.SetTracerProvider(tp)
  otel.SetTextMapPropagator(propagation.NewCompositeTextMapPropagator(
    propagation.TraceContext{}, propagation.Baggage{}))
  otel.SetErrorHandler(otel.ErrorHandlerFunc(func(err error) {
    logx.Errorf("[otel] error: %v", err)
  }))

  return nil
}
```

默认支持jaeger、zinpink

```go
package trace

// TraceName represents the tracing name.
const TraceName = "go-zero"

// A Config is a opentelemetry config.
type Config struct {
  Name     string  `json:",optional"`
  Endpoint string  `json:",optional"`
  Sampler  float64 `json:",default=1.0"`
  Batcher  string  `json:",default=jaeger,options=jaeger|zipkin"`
}
```

我们只需要在我们的业务代码配置中，也就是你的业务配置的yaml中配置参数即可。

## 2、实现

go-zero-looklook是以jaeger来实现的

### 2.1 jaeger

项目的docker-compose-env.yaml中配置了jaeger

```yaml
services:
  #jaeger链路追踪
  jaeger:
    image: jaegertracing/all-in-one:latest
    container_name: jaeger
    ports:
      - "5775:5775/udp"
      - "6831:6831/udp"
      - "6832:6832/udp"
      - "5778:5778"
      - "16686:16686"
      - "14268:14268"
      - "9411:9411"
    environment:
      - SPAN_STORAGE_TYPE=elasticsearch
      - ES_SERVER_URLS=http://elasticsearch:9200
      - LOG_LEVEL=debug
    networks:
      - looklook_net
      
   ........
```

其中jager_collector 依赖elasticsearch做存储，所以要把elasticsearch安装上，前一节收集日志时候我们已经演示了。

### 2.2 业务配置

我们以用户服务为例

1）api配置

app/usercenter/cmd/api/etc/usercenter.yaml

```yaml
Name: usercenter-api
Host: 0.0.0.0
Port: 8002
Mode: dev
......

#链路追踪
Telemetry:
  Name: usercenter-api
  Endpoint: http://jaeger:14268/api/traces
  Sampler: 1.0
  Batcher: jaeger
```

2）rpc配置

```yaml
Name: usercenter-rpc
ListenOn: 0.0.0.0:9002
Mode: dev

.....

#链路追踪
Telemetry:
  Name: usercenter-rpc
  Endpoint: http://jaeger:14268/api/traces
  Sampler: 1.0
  Batcher: jaeger
```

### 2.3 查看链路

请求用户服务注册、登陆、获取登陆用户信息

浏览器输入 [http://127.0.0.1:16686/search](https://link.segmentfault.com/?enc=j%2BpCncJe3XBNFARpigZVVg%3D%3D.j%2FGSjUkpKBQZKML%2FQwZ2%2FsLlI0utW%2B2DCmM7EYm0VNI%3D)即可查看

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041798114.png)

## 3、小结

日志、链路追踪我们都整理完了，好的系统一定能及时监控异常，接下来要看服务监控了。

# [第十三篇、服务监控](https://segmentfault.com/a/1190000041805576)

## 1、概述

好的服务一定是可以被及时监控的，在go-zero-looklook中我们使用目前比较流行的prometheus来作为监控工具，然后使用grafana来显示

go-zero已经在代码中给我们集成好了prometheus

```go
// StartAgent starts a prometheus agent.
func StartAgent(c Config) {
  if len(c.Host) == 0 {
    return
  }

  once.Do(func() {
    enabled.Set(true)
    threading.GoSafe(func() {
      http.Handle(c.Path, promhttp.Handler())
      addr := fmt.Sprintf("%s:%d", c.Host, c.Port)
      logx.Infof("Starting prometheus agent at %s", addr)
      if err := http.ListenAndServe(addr, nil); err != nil {
        logx.Error(err)
      }
    })
  })
}
```

无论当我们启动api、rpc都会额外启动一个goroutine 提供prometheus的服务

【注】如果像我们之前order-mq这种使用serviceGroup管理的服务，在启动文件main中要显式调用一下才可以，api、rpc不需要，配置都一样

```go
package main
.....
func main() {
  ....
  // log、prometheus、trace、metricsUrl.
  if err := c.SetUp(); err != nil {
    panic(err)
  }

  ......
}
```

## 2、实现

### 2.1 配置prometheus与grafana

在项目下的docker-compose-env.yml文件中

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041805578.png)

我们来deploy/prometheus/server/prometheus.yml看看prometheus配置文件

```yaml
global:
  scrape_interval:
  external_labels:
    monitor: 'codelab-monitor'

# 这里表示抓取对象的配置
scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s  #重写了全局抓取间隔时间，由15秒重写成5秒
    static_configs:
      - targets: ['127.0.0.1:9090']

  - job_name: 'order-api'
    static_configs:
      - targets: ['order-api:9091']
        labels:
          job: order-api
          app: order-api
          env: dev
  - job_name: 'order-rpc'
    static_configs:
      - targets: ['order-rpc:9091']
        labels:
          job: order-rpc
          app: order-rpc
          env: dev
  - job_name: 'order-mq'
    static_configs:
      - targets: ['order-mq:9091']
        labels:
          job: order-mq
          app: order-mq
          env: dev
  - job_name: 'usercenter-api'
    static_configs:
      - targets: ['usercenter-api:9091']
        labels:
          job: usercenter-api
          app: usercenter-api
          env: dev
  - job_name: 'usercenter-rpc'
    static_configs:
      - targets: ['usercenter-rpc:9091']
        labels:
          job: usercenter-rpc
          app: usercenter-rpc
          env: dev
  - job_name: 'travel-api'
    static_configs:
      - targets: ['travel-api:9091']
        labels:
          job: travel-api
          app: travel-api
          env: dev
  - job_name: 'travel-rpc'
    static_configs:
      - targets: ['travel-rpc:9091']
        labels:
          job: travel-rpc
          app: travel-rpc
          env: dev
  - job_name: 'payment-api'
    static_configs:
      - targets: ['payment-api:9091']
        labels:
          job: payment-api
          app: payment-api
          env: dev
  - job_name: 'payment-rpc'
    static_configs:
      - targets: ['payment-rpc:9091']
        labels:
          job: payment-rpc
          app: payment-rpc
          env: dev
  - job_name: 'mqueue-rpc'
    static_configs:
      - targets: ['mqueue-rpc:9091']
        labels:
          job: mqueue-rpc
          app: mqueue-rpc
          env: dev
  - job_name: 'message-mq'
    static_configs:
      - targets: ['message-mq:9091']
        labels:
          job: message-mq
          app: message-mq
          env: dev
  - job_name: 'identity-api'
    static_configs:
      - targets: ['identity-api:9091']
        labels:
          job: identity-api
          app: identity-api
          env: dev
  - job_name: 'identity-rpc'
    static_configs:
      - targets: [ 'identity-rpc:9091' ]
        labels:
          job: identity-rpc
          app: identity-rpc
          env: dev
```

### 2.2 业务配置

实现上我们业务也不需要添加任何代码（除了serviceGroup管理的服务）

我们只需要在业务配置文件中配置即可，我们拿usercenter来举例

1）api

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041805579.png)

2）rpc

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041805580.png)

3）mq（serviceGroup）

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041805581.png)

【注】（再强调一次）如果像我们之前order-mq这种使用serviceGroup管理的服务，在启动文件main中要显示调用一下才可以，api、rpc不需要

```go
package main
.....
func main() {
  ....
  // log、prometheus、trace、metricsUrl.
  if err := c.SetUp(); err != nil {
    panic(err)
  }

  ......
}
```

### 2.3 查看

访问 [http://127.0.0.1:9090/](https://link.segmentfault.com/?enc=DYBWtt%2FbaqU74uck0QQNLA%3D%3D.m8DrtnNEK4Y7SSkKHL009kBjMBndSBayaukgvpn7FTA%3D) ， 点击上面菜单“Status”，再点击Targets ,蓝色的就是启动成了，红色就是没启动成功

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041805582.png)

##### 2.4 配置grafana

访问[http://127.0.0.1:3001](https://link.segmentfault.com/?enc=vczRs2qWvYNmKnq5q%2Baunw%3D%3D.AAbiop4qdHYxuf9T%2B1MvDIqQ6V2AOPwLvEjEv4k54wg%3D)， 默认账号、密码都是admin

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041805583.png)

配置数据源是prometheus

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041805584.png)

然后配置

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041805585.png)

【注】这里是在docker中配置的，所以http的url不能写127.0.0.1

查看是否配置成功

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041805586.png)

配置dashboard

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041805587.png)

然后点击第一个

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041805588.png)

我们添加一个cpu指标，在下方输入cpu选择

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041805589.png)

然后我们就可以看到我们想要看的监控指标

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041805590.png)

## 3、结尾

这里只演示了一个指标，其他想看的指标自己配置就可以了，同时你也可以在grafana添加alert报警配置，这个就不作为演示了自行整理

# [第十四篇、部署环境搭建](https://segmentfault.com/a/1190000041844053)

## 1、概述

项目开发好后，我们需要部署，我们接下来就基于gitlab + jenkins + harbor + k8s 搭建部署环境

- gitlab: 放代码，可以做ci
- jenkins: 做cd发布项目
- harbor: 镜像仓库
- k8s: 运行服务

我们只在k8s内部运行服务，至于中间件(mysql、redis、es等)就会部署在k8s之外，如果你是线上使用云服务可以直接使用云服务，如果自建也最好运行在k8s之外。由于我是在本地演示，这里的中间件我就使用之前开发环境的中间件了，不要纠结这个，主要演示如何部署go-zero服务到k8s中

k8s部署这里就不介绍了，如果没有k8s环境自己用rancher或者kubeadm等搭建即可，再不行就去买个按时付费的云服务k8s

所以我们需要配置如下：

| 服务器名称        | 作用                                                         | Ip            |
| ----------------- | ------------------------------------------------------------ | ------------- |
| deploy-server.com | 部署gitlab、jenkins、harbor（预先装好docker、docker-compose） | 192.168.1.180 |
| srv-data.com      | 部署mysql、redis、es等等，模拟独立环境,k8s内部连接到此服务器 | 192.168.1.181 |
| nginx-gateway.com | 网关，独立于k8s集群外部                                      | 192.168.1.182 |
| k8s集群           | K8s 集群                                                     | 192.168.1.183 |

## 2、gitlab

### 2.1 部署gitlab

创建文件夹

```shell
$ mkdir gitlab && cd gitlab
$ vim docker-compose.yml
```

docker-compose.yml

```yaml
version: '3'

services:
    gitlab:
      image: 'twang2218/gitlab-ce-zh'
      container_name: 'gitlab'
      restart: always
      hostname: '192.168.1.180' #部署机器的ip,非容器ip(因为是本地不是线上所以用ip，线上的话可以用域名)
      environment:
        TZ: 'Asia/Shanghai'
        GITLAB_OMNIBUS_CONFIG: |
         external_url 'http://192.168.1.180'  #使用这个地址访问gitlab web ui(因为是本地不是线上所以用ip，线上的话可以用域名)
         gitlab_rails['gitlab_shell_ssh_port'] = 2222 #ssh clone代码地址
         unicorn['port'] = 8888 #gitlab一个内部端口
      ports:
        - '80:80'        #web 80 端口
       #- '443:443'      #web 443 端口,本次未使用就不开放了
        - '2222:22'      #ssh 检出代码 端口
      volumes:
        - ./etc:/etc/gitlab             #Gitlab配置文件目录
        - ./data:/var/opt/gitlab  #Gitlab数据目录
        - ./logs:/var/log/gitlab   #Gitlab日志目录
```

执行

```shell
$  docker-compose up -d
```

这个执行时间可能稍微有点长，不妨你可以去泡一杯coffee休息一下～～

### 2.2 访问gitlab

访问 [http://192.168.1.103](https://link.segmentfault.com/?enc=0NaVGhO6MA2INb9pbWJuOA%3D%3D.LpBjmTxDPIumy%2FqvM7X%2BXJZ4gcCVeHnU5k1kqebK5HY%3D)（即http://"docker-compose中ip/域名"）

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844055.png)

首次设置新密码： 12345678

账号默认是root

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844056.png)

### 2.3 创建项目

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844057.png)

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844058.png)

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844059.png)

### 2.4 配置ssh公钥

点击头像位置下箭头，“设置”

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844060.png)

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844061.png)

将自己的公钥配置上，点击“Add key”即可 （公钥不会生成的自己搜索，这里不详细说了）

### 2.5 上传项目

再点击项目，回到刚才创建的项目，将go-zero-looklook项目上传到此仓库ssh://git@192.168.1.180:2222/root/go-zero-looklook.git即可，到此我们的gitlab搭建就结束了。

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844062.png)

【注】 本次不做gitlab-runner演示，后续如果有时间再补充。

## 3、harbor

### 3.1 部署harbor

下载harbo [https://github.com/goharbor/h...](https://link.segmentfault.com/?enc=vSHepbdqSVI%2BF9e9fUFz%2Fg%3D%3D.p9seyHHYiQ%2BCbr7jT3s7ft5lhXLoNX%2B7X1MqnREMwikAdoonRa2CHyC48p4OBzjt3Vv6owiLTh9xbxpbMRXB31ct146DjEx7FESygfUjhf%2B%2BJcT4K753czJakS2AUFJA)，下载离线的offline安装会快点

下载解压后进入harbor文件夹

```shell
$ cd harbor && cp harbor.yml.tmpl harbor.yml
```

我们打开harbor.yml，修改如下

```yaml
hostname: 192.168.1.180                                                                       #修改为本机ip，不能使用localhost、127.0.0.1

http:
  port: 8077                                                                                              #改一下http端口8077

#https:                                                                                                      #暂时将https注释掉，我们先不通过https只通过http
#  port: 443
#  certificate: /your/certificate/path
#  private_key: /your/private/key/path

data_volume: /root/harbor/data          #修改一下数据目录位置

log:
  level: info
  local:
    rotate_count: 50
    rotate_size: 200M
    location: /root/harbor/log     #修改一下日志目录位置
```

直接运行 “sudo ./install.sh” 稍作等待。

### 3.2 访问harbor

浏览器输入 [http://192.168.1.180:8077](https://link.segmentfault.com/?enc=hyV2rqzHdbX7GRQXlllKdQ%3D%3D.WItWflJ37ewM1P5cA2h94yt0drivkw15PYRgpikB2YU%3D)

账号: admin

密码: Harbor12345 (在harbor.yml中记录的，默认是Harbor12345)

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844063.png)

登陆成功

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844064.png)

##### 3.3 新建私有项目

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844065.png)

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844066.png)

查看push命令

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844067.png)

```shell
$ docker push 192.168.1.180:8077/go-zero-looklook/REPOSITORY[:TAG]
```

### 3.4支持http

默认拉取、推送到镜像仓库要使用https , 由于我们这里没有https，需要用http , 所以我们要在deploy-server.com服务器上执行如下

```shell
$ echo '{"insecure-registries":["192.168.1.180:8077"] }' >> /etc/docker/daemon.json
```

到此我们harbor搭建完成。

## 4、jenkins

### 4.1 部署jenkins

创建文件夹

```shell
$ mkdir jenkins && cd jenkins
$ vim docker-compose.yml
```

docker-compose.yml

```yaml
version: '3'
services:
  jenkins:
    image: 'jenkins/jenkins:lts'
    container_name: jenkins
    restart: always
    environment:
      - TZ=Asia/Shanghai
    user: root
    ports:
      - '8989:8080'
      - '50000:50000'
    volumes:
      - './jenkins_home:/var/jenkins_home'
      - '/var/run/docker.sock:/var/run/docker.sock'
      - '/usr/bin/docker:/usr/bin/docker'
      - '/root/port.sh:/root/port.sh'
```

【注】/root/port.sh内容如下

```bash
#!/bin/sh

case $1 in
"identity-api") echo 1001
;;
"identity-rpc") echo 1101
;;
"usercenter-api") echo 1002
;;
"usercenter-rpc") echo 1102
;;
"message-mq") echo 1207
;;
"mqueue-rpc") echo 1106
;;
"order-api") echo 1004
;;
"order-mq") echo 1204
;;
"order-rpc") echo 1104
;;
"payment-api") echo 1005
;;
"payment-rpc") echo 1105
;;
"travel-api") echo 1003
;;
"travel-rpc") echo 1103
esac
```

执行

```shell
$ docker-compose up -d
```

这个时间也不慢，可以再去喝一杯coffee

### 4.2 挂载工具

1）将goctl 复制到 jenkins容器中

```shell
$ docker cp $GOPATH/bin/goctl jenkins:/usr/local/bin
$ docker exec -it jenkins /bin/sh #进入jenkins 容器
$ goctl -v     #验证成功
goctl version 1.3.0-20220201 linux/amd64
```

2）将kubectl文件复制到jenkins容器中

```shell
$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
$ sudo chmod a+x kubectl
$ docker cp kubectl jenkins:/usr/local/bin
$ docker exec -it jenkins /bin/sh #进入jenkins 容器
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.3" .....
```

3）将k8s的配置.kube/config复制到jenkins容器

```shell
$ docker cp ~/.kube jenkins:/root/ #前提是家目录下的.kube文件夹中存在k8s的config配置
$ docker exec -it jenkins /bin/sh #进入jenkins 容器
$ kubectl ge ns
default              Active   43m
kube-node-lease      Active   43m
kube-public          Active   43m
kube-system          Active   43m
local-path-storage   Active   43m
```

【注】上面这四步，也可以直接打进镜像中，我这里只是演示，留给你们自己处理。

### 4.3 访问jenkins

[http://192.168.1.180:8989](https://link.segmentfault.com/?enc=yEM9iDOUcTBZIfGR05P0Mg%3D%3D.tOF7WaMvoOxkJJmBMq4X7xqFzCISm5KeRK4NjqQ61ug%3D)

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844068.png)

第一次访问出现上面图不要慌，让你稍等一会，它在进行准备工作，准备好后会自动跳到登陆页面。

出现如下界面，说明准备好了，因为我们目录是挂载出来的，我们查看本机jenkins_home/secrets/initialAdminPassword密码，输入下一步即可

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844069.png)

选择“安装推荐插件“

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844070.png)

然后等待插件安装完成

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844071.png)

### 4.4 创建用户

账号：root/root

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844072.png)

### 4.5 部署完成

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844073.png)

到此 jenkins 部署完成

### 4.6 添加凭据

点击左边菜单“Manage Jenkins”

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844074.png)

点击 "Manage Credentials"

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844075.png)

点击“全局”后面的三角标，然后再点击“添加凭据”

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844076.png)

进入“添加凭据”页面，类型我们选择 “SSH Username with private key” 使用私钥方式，`Username`是gitlab一个标识，后面添加pipeline你知道这个标识是代表gitlab的凭据自行定义的，`Private Key`即在gitlab配置的私钥（之前我们配置在gitlab的公钥对应的私钥，在这里就是我们自己本机的私钥），我们这个凭证就是给jenkins用来去gitlab时候免密拉代码用的

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844077.png)

确定即可。

### 4.7 添加harbor仓库配置

进入首页，点击左侧菜单`Manage Jenkins`->`Configure System`

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844078.png)

下滑动到`全局属性`条目，添加docker私有仓库相关信息，如图为`docker用户名`、`docker用户密码`、`docker私有仓库地址`

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844079.png)

点击 “保存”

### 4.8 配置git

进入`Manage Jenkins`->`Global Tool Configureation`，找到Git条目，填写jenkins所在机器git可执行文件所在path，如果没有的话，需要在jenkins插件管理中下载Git插件, 有就不需要管了（如下图）

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844080.png)

配置pipline需要的Git Parameter插件

点击 “系统配置” -> “插件管理”

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844081.png)

然后点击“可选插件” , 搜索中输入 “Git Parameter” , 如下图

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041844082.png)

安装好，重启后即可，到此jenkins搭建完成。

## 5、k8s

k8s的部署这里就不介绍了，自己用kubeadm、rancher、kind去安装吧，或者买个按量云容器服务，总之有一个k8s集群就好了。

# [大结局篇、k8s部署](https://segmentfault.com/a/1190000041854213)

## 1、概述

上一节，我们已经把gitlab、jenkins、harbor、k8s都已经搭建好了，这一节我们来编写jenkins的pipline将我们的服务通过jenkins完整的发布到k8s中。

## 2、部署中间件

将mysql、redis、es等部署到k8s之外 ， 模拟用作线上独立环境（至于线上你想把某些中间件部署到k8s内部这个自行处理，本次重点是如何将go-zero开发的微服务部署到k8s集群内部），这里我就直接使用项目下的docker-compose-env.yaml了，把所有依赖的第三方中间件环境直接安装在srv-data.com(192.168.1.181)这台服务器，前提是这台服务器已经安装好docker、docker-compose。

登陆到 192.168.1.181

```shell
$ mkdir data && cd data && vim docker-compose.yml
$ docker-compose up -d
$ docker-compose ps #查看确认
```

## 3、独立配置

将每个服务的配置都独立出来，统一放在一个git仓库，这样只给一个人线上仓库的权限，如果线上配置有变直接修改这个仓库的文件，在jenkins做cd的时候，会先拉取代码再拉取对应服务的配置自动构建，具体可以看后面的pipline。

【问】为什么不用配置中心？

1）修改db、redis等需要重启服务，但是有一些配置又不需要重启服务，运维又要去记，记混了比较容易造成线上事故

2）方便回滚。我们发新版本到线上，并且又改了新版本配置。这时候线上用户反馈有问题，线上需要快速回滚的话，如果我们使用将文件构建到镜像中，直接使用k8s一行命令就可以将上一个版本代码加配置直接回滚回来。如果使用了配置中心，回滚了代码，还要将上个版本的配置去配置中心改回来，很麻烦。

独立线上仓库目录结构如下（这个结构是跟pipline中写法相关的）

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041854215.png)

仓库地址 ： [https://github.com/Mikaelemmm...](https://link.segmentfault.com/?enc=r2NOq4GUhR68GcdvdcHu3Q%3D%3D.aOfWk%2FIgsVq03X4SC%2F4KWaO7heTTyHlqOa%2BH6mQk4fbpz7Dl1skB38FLeIAy7sheo0Dw9y%2BeS%2B1xFtZv2Iy1sw%3D%3D) , 直接下载就好

> 1、修改配置中的中间件，数据库、redis等都要改成192.168.1.181这台机器，我们把这台机器当成线上环境的中间件。
>
> 2、另外一个就是我们的服务发现，线上我们部署在k8s中，go-zero直接支持k8s服务发现，所以不需要etcd等，我们在配置zrpc client的时候，要改成target，k8s的配置方式。

## 4、编写 jenkins 的 pipline

### 4.1 配置参数

访问 [http://192.168.1.180:8989/](https://link.segmentfault.com/?enc=zEfrENAA%2FUTacNoGFuZJ%2Fw%3D%3D.2fJa2UCTFQT9JN6oPJOmXes1QMDRTnd6ona3QkloMS0%3D) 打开jenkins，进入jenkins首页，点击左侧菜单`新建Item`

我们先创建 `identity` 授权服务的流水线

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041854216.png)

然后点击“General” , 选择“This project is parameterized” ， "添加参数"，“Choice Parameter”，如下图

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041854217.png)

然后编写内容如下

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041854218.png)

直接保存。

### 4.2 编写pipline

向下滑动找到`Pipeline script`,填写脚本内容

```nsis
pipeline {
  agent any
  parameters {
      gitParameter name: 'branch', 
      type: 'PT_BRANCH',
      branchFilter: 'origin/(.*)',
      defaultValue: 'master',
      selectedValue: 'DEFAULT',
      sortMode: 'ASCENDING_SMART',
      description: '选择需要构建的分支'
  }

  stages {
      stage('服务信息')    {
          steps {
              sh 'echo 分支：$branch'
              sh 'echo 构建服务类型：${JOB_NAME}-$type'
          }
      }

      stage('拉取代码') {
          steps {
              checkout([$class: 'GitSCM', 
              branches: [[name: '$branch']],
              doGenerateSubmoduleConfigurations: false, 
              extensions: [], 
              submoduleCfg: [],
              userRemoteConfigs: [[credentialsId: 'gitlab-cert', url: 'ssh://git@192.168.1.180:2222/root/go-zero-looklook.git']]])
          }   
      }

      stage('获取commit_id') {
          steps {
              echo '获取commit_id'
              git credentialsId: 'gitlab-cert', url: 'ssh://git@192.168.1.180:2222/root/go-zero-looklook.git'
              script {
                  env.commit_id = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
              }
          }
      }

      stage('拉取配置文件') {
              steps {
                  checkout([$class: 'GitSCM', 
                  branches: [[name: '$branch']],
                  doGenerateSubmoduleConfigurations: false, 
                  extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'conf']], 
                  submoduleCfg: [],
                  userRemoteConfigs: [[credentialsId: 'gitlab-cert', url: 'ssh://git@192.168.1.180:2222/root/go-zero-looklook-pro-conf.git']]])
              }   
      }

      stage('goctl版本检测') {
          steps{
              sh '/usr/local/bin/goctl -v'
          }
      }
      
      stage('Dockerfile Build') {
          steps{
                 sh 'yes | cp  -rf conf/${JOB_NAME}/${type}/${JOB_NAME}.yaml  app/${JOB_NAME}/cmd/${type}/etc'   //线上配置文件
                 sh 'cd app/${JOB_NAME}/cmd/${type} && /usr/local/bin/goctl docker -go ${JOB_NAME}.go && ls -l'
                 script{
                     env.image = sh(returnStdout: true, script: 'echo ${JOB_NAME}-${type}:${commit_id}').trim()
                 }
                 sh 'echo 镜像名称：${image} && cp app/${JOB_NAME}/cmd/${type}/Dockerfile ./  && ls -l && docker build  -t ${image} .'
          }
      }

      stage('上传到镜像仓库') {
          steps{
                //docker login 这里要注意，会把账号密码输出到jenkins页面，可以通过port.sh类似方式处理，官网文档有这里我就不详细写了
              sh 'docker login --username=${docker_username} --password=${docker_pwd} http://${docker_repo}' 
              sh 'docker tag  ${image} ${docker_repo}/go-zero-looklook/${image}'
              sh 'docker push ${docker_repo}/go-zero-looklook/${image}'
          }
      }

      stage('部署到k8s') {
          steps{
              script{
                  env.deployYaml = sh(returnStdout: true, script: 'echo ${JOB_NAME}-${type}-deploy.yaml').trim()
                  env.port=sh(returnStdout: true, script: '/root/port.sh ${JOB_NAME}-${type}').trim()
              }

              sh 'echo ${port}'

              sh 'rm -f ${deployYaml}'
              sh '/usr/local/bin/goctl kube deploy -secret docker-login -replicas 2 -nodePort 3${port} -requestCpu 200 -requestMem 50 -limitCpu 300 -limitMem 100 -name ${JOB_NAME}-${type} -namespace go-zero-looklook -image ${docker_repo}/${image} -o ${deployYaml} -port ${port} --home /root/template'
              sh '/usr/local/bin/kubectl apply -f ${deployYaml}'
          }
      }

       stage('Clean') {
           steps{
               sh 'docker rmi -f ${image}'
               sh 'docker rmi -f ${docker_repo}/${image}'
               cleanWs notFailBuild: true
           }
       }
  }
}
```

> 非常重要！！！

1. 构建优化：pipline中生成dockerfile的时候，我们是使用k8s方式部署不需要etcd，但是这种方式部署需要指定账号（有去k8s的endpoints中get的权限，使用默认default就好了，每次创建一个新的命名空间k8s会自动帮我们创建好一个default），但是使用goctl 生成的 k8s yml没有添加指定账号选项，这个已经反馈了，可能后续版本会加上，这里我们也用模版做了，同样模版是在项目目录下[https://github.com/Mikaelemmm...](https://link.segmentfault.com/?enc=WoUSASq6ycdggEi2sN3G6g%3D%3D.ikv%2BYm%2B8k5iSVKNDVsZDPVNWUoVkxye8VpQpRb85H%2BAWoNK23mcRHdIv5EofjZiLekfyl808rrsZ1kEUtriyUN4tHGS%2BPjv%2Bzp%2BVFzXm9lw%3D)，pipline中构建指定这个模版即可
2. ${credentialsId}要替换为你的具体凭据值，即【添加凭据】模块中的一串字符串，我们之前配置的是gitlab-cert所以这里就填写gitlab-cert，如果你不是这个自己要更换，${gitUrl}需要替换为你代码的git仓库地址，其他的${xxx}形式的变量无需修改，保持原样即可。
3. 这里跟官方文档有一点点不一样，由于我项目文件夹目录不同，goctl生成的dockerfile文件我手动做了点调整，在一个我不是在构建时候生成的dockerfile，是在创建项目时候就把dockerfile一起放在目录下，这样构建镜像时候不需要goctl了

## 5、配置k8s拉取私有仓库镜像

k8s在默认情况下，只能拉取harbor镜像仓库的公有镜像，如果拉取私有仓库镜像，则是会报 `ErrImagePull` 和 `ImagePullBackOff` 的错误

1、先在jenkins发布机器登陆harbor

```shell
$ docker login 192.168.1.180:8077
$ Username: admin
$ Password:
Login Succeeded
```

2、在k8s中生成登陆harbor配置文件

```shell
#查看上一步登陆harbor生成的凭证
$ cat /root/.docker/config.json  
{
    "auths": {
        "192.168.1.180:8077": {
            "auth": "YWRtaW46SGFyYm9yMTIzNDU="
        }
}
```

3、对秘钥文件进行base64加密

```shell
$ cat /root/.docker/config.json  | base64 -w 0

ewoJImF1dGhzIjogewoJCSIxOTIuMTY4LjEuMTgwOjgwNzciOiB7CgkJCSJhdXRoIjogIllXUnRhVzQ2U0dGeVltOXlNVEl6TkRVPSIKCQl9Cgl9Cn0=
```

4、创建docker-secret.yaml

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: docker-login
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: ewoJImF1dGhzIjogewoJCSIxOTIuMTY4LjEuMTgwOjgwNzciOiB7CgkJCSJhdXRoIjogIllXUnRhVzQ2U0dGeVltOXlNVEl6TkRVPSIKCQl9Cgl9Cn0=
$ kubectl create -f docker-secret.yaml -n go-zero-looklook

secret "docker-login" created
```

## 6、构建

我们进入首页，点击idenity进入详情页

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041854219.png)

然后可以看到，上面我们配置好的identity服务，如下图 ，点击“Build with Parameters”， 然后选择rpc,点击“开始构建”

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041854220.png)

> 第一次构建在拉代码时候都会失败，应该是初始化啥东西，再点一次就好了。

部署成功

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041854222.png)

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041854223.png)

![img](微服务从代码到k8s部署应有尽有系列.assets/1460000041854224.png)

同样道理，去构建identity-api，再去配置usercenter服务 构建usercenter-rpc、构建usercenter-api，接着配置其他服务、构建即可，本次我们先只构建identity-api、identity-rpc、usercenter-rpc、usercenter-api给大家演示。

#### 6、添加网关

因为我们的api服务通过goctl发布在k8s中都会暴露nodeport端口，索引我们看下k8s中go-zero-looklook命名空间下的service的nodeport端口服务，然后将nodeport配置在nignx即可。

本次我们独立一台虚拟机在k8s之外，安装nginx，将k8s后端api服务通过nodeport方式把端口暴露给nginx，然后nginx在配置中配置此api服务，这样nginx就充当网关使用。

nginx的安装就不再这里多说了，记得一定要有auth_request模块，没有的话自己去安装。

nginx的配置

```nginx
server{
    listen 8081;
    access_log /var/log/nginx/looklook.com_access.log;
    error_log /var/log/nginx//looklook.com_error.log;

    location /auth {
        internal;
      proxy_set_header X-Original-URI $request_uri;
        proxy_pass_request_body off;
        proxy_set_header Content-Length "";
        proxy_pass http://192.168.1.182:31001/identity/v1/verify/token;
    }

    location ~ /usercenter/ {
       auth_request /auth;
       auth_request_set $user $upstream_http_x_user;
       proxy_set_header x-user $user;

       proxy_set_header Host $http_host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header REMOTE-HOST $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_pass http://192.168.1.182:31002;
   }

   location ~ /travel/ {
       auth_request /auth;
       auth_request_set $user $upstream_http_x_user;
       proxy_set_header x-user $user;

       proxy_set_header Host $http_host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header REMOTE-HOST $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_pass http://192.168.1.182:31003;
   }

   location ~ /order/ {
       auth_request /auth;
       auth_request_set $user $upstream_http_x_user;
       proxy_set_header x-user $user;

       proxy_set_header Host $http_host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header REMOTE-HOST $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_pass http://192.168.1.182:31004;
   }

   location ~ /payment/ {
       auth_request /auth;
       auth_request_set $user $upstream_http_x_user;
       proxy_set_header x-user $user;

       proxy_set_header Host $http_host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header REMOTE-HOST $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_pass http://192.168.1.182:31005;
   }
}
```

如果是线上的话，应该配置多台nignx保持高可用，在nignx前面还会有一个slb，你的域名包括https配置都应该解析到slb，在slb前面在有防火墙等这些。

## 8、结束语

至此，整个系列就结束了，整体架构图应该如第一篇所展示，本系列希望能给你带来帮助。