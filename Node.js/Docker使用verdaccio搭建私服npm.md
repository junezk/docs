# Docker使用verdaccio搭建私服npm

一、场景和需求分析
搭建公司的私有npm服务器，对于带宽要求最小5M。

二、部署方式

```
#环境准备
docker pull verdaccio/verdaccio
mkdir -p /opt/docker/verdaccio/conf  #config.yaml和htpasswd存放在这里
mkdir -p /opt/docker/verdaccio/storage #包文件等
chown -R 100:101 /opt/docker/verdaccio/ #给容器对物理目录的读写权限
```

将以下内容写入config.yaml
$ sudo cat /opt/docker/verdaccio/conf/config.yaml

```yaml
# path to a directory with all packages
storage: /verdaccio/storage

auth:
  htpasswd:
    file: /verdaccio/conf/htpasswd
    # Maximum amount of users allowed to register, defaults to "+infinity".
    # You can set this to -1 to disable registration.
    #max_users: 1000

# a list of other known repositories we can talk to
uplinks:
  npmjs:
    url: https://registry.npm.taobao.org/

packages:
  '@*/*':
    # scoped packages
    access: $all
    publish: $authenticated
    proxy: npmjs

  '**':
    # allow all users (including non-authenticated users) to read and
    # publish all packages
    #
    # you can specify usernames/groupnames (depending on your auth plugin)
    # and three keywords: "$all", "$anonymous", "$authenticated"
    access: $all

    # allow all known users to publish packages
    # (anyone can register by default, remember?)
    publish: $authenticated

    # if package is not available locally, proxy requests to 'npmjs' registry
    proxy: npmjs

# To use `npm audit` uncomment the following section
middlewares:
  audit:
    enabled: true

# log settings
logs:
  - {type: stdout, format: pretty, level: http}
```

```shell
#启动启动容器
docker run -it  --name verdaccio -p 4873:4873   -v /opt/docker/verdaccio/conf:/verdaccio/conf   -v /opt/docker/verdaccio/storage:/verdaccio/storage   verdaccio/verdaccio

#后台启动只需要把-it改为-d
```

参考链接：
https://www.verdaccio.org/docs/en/docker.html
https://github.com/verdaccio/verdaccio/blob/master/Dockerfile
https://segmentfault.com/a/1190000013469391
http://ju.outofmemory.cn/entry/338244