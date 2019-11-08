# Vue项目部署到Nginx

# 下载Nginx

以windows版为例，下载niginx压缩包并解压到任意目录，双击`nginx.exe`，在浏览器中访问`http://localhost`，如果出现`Welcome to nginx!`页面则说明成功。

## nginx常用命令如下：

```
nginx -h        # 打开帮助
nginx -t        # 检查配置文件是否正确

# 以下命令均要先启动nginx才能执行
nginx -s stop   # 停止
nginx -s quit   # 退出
nginx -s reopen # 重新启动（注意不会重新读取配置文件）
nginx -s reload # 重新读取配置文件
```

# 部署项目到Nginx根目录

对于`vue-cli`创建的项目，修`改vue.config.js`文件（位于项目根目录下，没有的话自行创建）

```
module.exports = {
  // 开发环境中使用的端口
  devServer: {
    port: 8001
  },
  // 取消生成map文件（map文件可以准确的输出是哪一行哪一列有错）
  productionSourceMap: false,
  // 开发环境和部署环境的路径
  publicPath: process.env.NODE_ENV === 'production'
    ? '/'
    : '/my/',
  configureWebpack: (config) => {
    // 增加 iview-loader
    config.module.rules[0].use.push({
      loader: 'iview-loader',
      options: {
        prefix: false
      }
    })
    // 在命令行使用 vue inspect > o.js 可以检查修改后的webpack配置文件
  }
}
```

在vue项目根目录中使用命令`npm run build`创建输出文件，将dist文件夹下的所有内容复制到nginx目录下的`webapp/`内（没有的话自行创建）。

修改nginx目录中的`conf/nginx.conf`文件，在 http -> server 节点中，修改location节的内容：

```
location / {
    root   webapp;
    index  index.html index.htm;
}
```

在nginx根目录使用命令`nginx -s reload`即可在浏览器中通过`http://localhost`访问项目。

# 多个项目部署到Nginx

有时一个Nginx中放了好几个子项目，需要将不同的项目通过不同的路径来访问。

对于项目1而言，修改`vue.config.js`文件的`publicPath`：

```
publicPath: '/vue1/'
```

对于项目2而言，修改`vue.config.js`文件的`publicPath`：

```
publicPath: '/vue2/'
```

分别在vue项目根目录中使用命令`npm run build`创建输出文件，将dist文件夹下的所有内容复制到nginx目录下的`webapp/vue1和webapp/vue2`内（没有的话自行创建）。

修改nginx目录中的`conf/nginx.conf`文件，在 http -> server 节点中，修改location节的内容：

```
location /vue1 {
    root   webapp;
    index  index.html index.htm;
}

location /vue2 {
    root   webapp;
    index  index.html index.htm;
}
```

在nginx根目录使用命令`nginx -s reload`即可在浏览器中通过`http://localhost/vue1、http://localhost/vue2`访问项目1、项目2。

# 端口代理

当前后端项目分别部署在同一台机器上时，由于无法使用相同的端口，故后端一般会将端口号设置成不同的值（例如8080），但是当前端向后端请求资源时还要加上端口号，未免显得麻烦，故利用可以nginx将前端的指定路径代理到后端的8080端口上。

在`conf/nginx.conf`文件中增加`location`：

```
location /api {
    proxy_pass http://localhost:8080/api;
}
```

这样，当前端访问`/api`路径时，实际上访问的是`http://localhost:8080/api`路径。

# 其他

- **将博客听歌视频于一体的综合性网站** [juejin.im/post/5da2a8…](https://juejin.im/post/5da2a8ed6fb9a04de818eeff)
- **7天撸完KTV点歌系统,含后台管理系统(完整版)** [juejin.im/post/5da2a8…](https://juejin.im/post/5dac3b4351882576534d33d7)
- **NVM自由切换Node版本小技巧** [juejin.im/post/5da2a8…](https://juejin.im/post/5dae55b75188257d8936be94)