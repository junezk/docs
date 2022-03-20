# npm私服搭建—verdaccio方案及其最佳实践

##### 为什么要搭建公司内部的npm服务？

如今前端圈已十分繁荣，一个跟得上技术时代脚步的互联网公司必定是前后端分离的（至少在狭义上是分离的），这就导致了前端承受的分工压力会越来越大，很多公司的前端项目变得十分庞杂，因此技术负责人可能会考虑根据业务线进行拆分为几个工程系统，这样又引申出一个问题：这几个系统之间如何共用一套公司内部的组件库呢？每个工程里面都copy一套肯定不行，做一个小修改要同步几个工程，繁琐且容易出错。上传到npm库是个很不错的选择，不同的系统都指向一个npm源，然后通过npm install就行，快速且“干净”，所以在这种场景下，搭建一套公司内部的npm库就显得很有必要。简单来说，npm私服主要优势其实就两个：

- 托管公司内部组件库代码，不对外，方便管理
- 项目中使用到的npm包会缓存到私服库里，能明显提升之后下包的速度

##### 为什么选择verdaccio

网上主要有几种搭建企业npm私服的方式，简单罗列一下：

1. 付费选择：

- MyGet ([https://www.myget.org](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.myget.org%2F)) 9美元/月，且只能有两个账号和1GB的存储空间。
- NPM Org ([https://www.npmjs.com](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.npmjs.com%2F)) 每个账号每月7美元

1. 免费选择：

- DIY NPM ([https://docs.npmjs.com/misc/registry](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.npmjs.com%2Fmisc%2Fregistry))
- Git，这也是一种选择，在package.json中指定git仓库的URL即可，但是这种做法有些别扭，第一，使得package.json不够优雅，第二，当git仓库为private时，你需要HTTPS或SSH凭据，而且通常我们并没有每个团队的权限。
- Sinopia ([https://www.npmjs.com/package/sinopia](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Fsinopia))
- Cnpmjs.org ([https://github.com/cnpm/cnpmjs.org](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fcnpm%2Fcnpmjs.org))

付费的我们就不考虑了，没这个必要，而且付费的也不是就更好。sinopia搭建十分简单友好，不过这玩意儿已经停止维护了，最近的更新在4年前，但有一群人出了sinopia的一个分支，起了个名字叫**verdaccio**，这个就是这次主要推荐的方案，这个库一直在积极维护中，github start 7000+，看来还是比较靠谱的，而且国内外各种资料参考下来，这个方案也是受到极力推荐的。verdaccio搭建私服很简单，相比于cnpm搭建，还需要安装配置mysql，这个绝对会少走一些坑。

![img](https:////upload-images.jianshu.io/upload_images/16557061-005e9531efb3da6e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

##### 搭建步骤

1. 首先我们需要向运维同学申请一台linux服务器，给台2GB左右的虚拟机就够用了；
2. 找个合适的地方下载安装nodejs，比如在`/usr/local/lib`下
    安装wget：`yum install -y wget`；（已经安装的跳过这步）
    下载：`wget https://nodejs.org/dist/v10.6.0/node-v10.6.0-linux-x64.tar.xz`;
    解压：`tar -xvf node-v10.6.0-linux-x64.tar.xz`;
    重命名安装目录：`mv node-v10.6.0-linux-x64 nodejs`;
    建立软连接：
    `ln -s /usr/local/lib/nodejs/bin/npm /usr/local/bin/`
    `ln -s /usr/local/lib/nodejs/bin/node /usr/local/bin/`
3. 执行`node -v`和 `npm -v`命令检查是否安装成功
4. 全局安装verdaccio：`npm i verdaccio -g`;
5. 全局安装pm2，用来守护node进程：`npm i pm2 -g`;
6. 安装nginx，仍然在 `/usr/local/lib`下
    下载：`wget http://nginx.org/download/nginx-1.13.7.tar.gz`
    解压：`tar -zxvf nginx-1.13.7.tar.gz`；
    换个名字：`mv nginx-1.13.7 nginx`；
    进入安装目录：`cd nginx`
    执行：`./configure`；
    执行：`make && make install`；
    `cd conf/`修改nginx.conf，加上这一段：

```nginx
server {
  listen       80;
  server_name  registry.npm.your.server;
  location / {
    proxy_pass    http://127.0.0.1:4873/;
    proxy_set_header        Host $host;
  }
}
```

建立软连接：`ln -s /usr/local/nginx/sbin/nginx /usr/local/bin/`
 启动nginx: `sudo nginx`
 （重启命令：`sudo nginx -s reload`）

1. pm2启动服务，执行

   ```
   pm2 start verdaccio
   ```

   ，然后浏览器访问

   ```
   http://服务器IP
   ```

   ，出现以下页面则代表安装成功。

   ![img](https:////upload-images.jianshu.io/upload_images/16557061-f279575c711bf5f2.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

   image.png

##### verdaccio使用方式

verdaccio允许任何人创建账号，若没有配置verdaccio的配置文件`config.yaml`，则默认任何注册了verdaccio的开发都有publish权限。看个实例：

1. 添加一个用户：`npm adduser --registry http://172.16.14.5`:

   ![img](https:////upload-images.jianshu.io/upload_images/16557061-f09037bee8d68702.png?imageMogr2/auto-orient/strip|imageView2/2/w/1066/format/webp)

2. 给要添加到服务的工程添加源信息，在工程根目录下新建`.npmrc`文件，添加以下内容：

```nginx
registry=http://172.16.14.5/
```

1. `package.json`中设置好版本，执行`npm publish`:

![img](https:////upload-images.jianshu.io/upload_images/16557061-2ddbea8a98eb43bd.png?imageMogr2/auto-orient/strip|imageView2/2/w/1050/format/webp)

此时再访问http://172.16.14.5，该工程已出现在列表中：

![img](https:////upload-images.jianshu.io/upload_images/16557061-883e8ebebba7b374.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

##### verdaccio最佳实践

上述publish仓库的步骤很简单，创建用户 -> 设置npm源 -> npm publish。但这样并不是我们想要的流程，我们想要的是publish权限可把控，这当然是可以做的。

**1. 查看config.yaml配置文件**
 一般来说此配置文件在`/创建用户/.config/verdaccio`中，如果你不确定，你可以直接在服务器上执行`verdaccio`命令，第一个config file即文件所在位置.

![img](https:////upload-images.jianshu.io/upload_images/16557061-46779f058a8eea72.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

这个目录下有两个文件及一个目录：

![img](https:////upload-images.jianshu.io/upload_images/16557061-9b518e2925293ca0.png?imageMogr2/auto-orient/strip|imageView2/2/w/536/format/webp)

image.png

打开`config.yaml`，查看packages项，原始内容为：

```bash
packages:
  '@*/*':
    # scoped packages
    access: $all
    publish: $authenticated
    unpublish: $authenticated
    proxy: npmjs

  '**':
    # allow all users (including non-authenticated users) to read and
    # publish all packages
    #
    # you can specify usernames/groupnames (depending on your auth plugin)
    # and three keywords: "$all", "$anonymous", "$authenticated"
    access: $all

    # allow all known users to publish/publish packages
    # (anyone can register by default, remember?)
    publish: $authenticated
    unpublish: $authenticated

    # if package is not available locally, proxy requests to 'npmjs' registry
    proxy: npmjs
```

**字段含义：**
 scope有两种模式
 一种是 @/ 表示某下属的某项目
 另一种是 * 匹配项目名称(名称在package.json中有定义)
 权限：

- `access`: 表示哪一类用户可以对匹配的项目进行安装(install)
- `publish`: 表示哪一类用户可以对匹配的项目进行发布(publish)
- `proxy`: 如其名，这里的值是对应于 uplinks 的名称，如果本地不存在，允许去对应的uplinks去取。

**值的含义：**

- `$all` 表示所有人(已注册、未注册)都可以执行对应的操作
- `$authenticated` 表示只有通过验证的人(已注册)可以执行对应操作，注意，任何人都可以去注册账户。
- `$anonymous` 表示只有匿名者可以进行对应操作（通常无用）

如果要指定某个用户才有权限，可以直接写上用户名，多个用户用空格隔开，比如：



```undefined
publish: michael martin
```

修改完成后请重启nginx和pm2。当前已存在的用户列表可在`htpasswd`文件中查看。

##### 下载包及缓存仓库

在指定源为内网源的工程下，通过`npm install`就能下载内网的库了。我们内部的仓库存储在`storage`目录里，进入`storage`中，可以看到这里存储了我们使用过的库：

![img](https:////upload-images.jianshu.io/upload_images/16557061-c6fe08b09a7eae1f.png?imageMogr2/auto-orient/strip|imageView2/2/w/774/format/webp)

其中testpro是我们内部的库，进入库中，可以看到我们发布的所有版本都在这里以压缩包的形式存储着

![img](https:////upload-images.jianshu.io/upload_images/16557061-a32f1c7e5755541e.png?imageMogr2/auto-orient/strip|imageView2/2/w/852/format/webp)

##### 参考

1. [Ways to have your private npm registry ](https://links.jianshu.com/go?to=https%3A%2F%2Fmedium.com%2Fengenharia-noalvo%2Fways-to-have-your-private-npm-registry-and-a-final-diy-solution-eed001a88e74)
2. [https://verdaccio.org/docs/en/what-is-verdaccio](https://links.jianshu.com/go?to=https%3A%2F%2Fverdaccio.org%2Fdocs%2Fen%2Fwhat-is-verdaccio)