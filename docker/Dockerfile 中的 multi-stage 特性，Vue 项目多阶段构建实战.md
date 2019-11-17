# Dockerfile 中的 multi-stage 特性，Vue 项目多阶段构建实战

最近在写一个 Vue 项目，并已经实现 Docker 镜像构建和容器化部署，之前也分享过一篇文章关于 Vue 项目的静态资源打包和镜像构建。但是之前在镜像构建之前是需要使用 npm 进行项目打包生产 dist 文件之后才能进行镜像构建，不过最近我在使用 Jenkins pipeline 的时候突然想到了 Dockerfile 的多阶段构建，完全可以使用这个特性直接构建出镜像。

## 多阶段构建场景

所谓的多阶段构建，就是当有的服务是需要编译环境进行编译或者打包，然后才能将构建产物移到运行环境中的多个阶段的构建形式。

比如这篇文章所提到的 Vue 项目的构建，Vue 的项目无论是简单还是复杂，最终都是会形成一个静态资源文件夹，这个文件夹里面的内容才是需要放到运行环境中使用的。同样的，Java 项目一般也是需要需要进行编译打包，形成各种包然后才会进行构建。

说的简单一点，就是当一个项目从项目代码到构建成 Docker 镜像的过程需要在不同的环境中进行的场景，那就属于多阶段构建。

## 多阶段构建实战

本篇文章就来分享一下我这个 Vue 项目的多阶段构建实战，其实这不仅仅能代表我这一个项目，而是可以代表所有 Vue 类型的项目的构建思路。

### 单阶段构建步骤

由于多阶段构建的依据其实也是单阶段构建，只是把多个步骤集中到一个 Dockerfile 里面而已，所以要实现多阶段构建，首先需要明确真个构建需要做的事情，理清步骤才能开始构建。

一个 Vue 项目从项目代码到形成镜像，主要分两个步骤，第一步是将源代码构建出静态资源文件，第二步就是选择基础镜像，将静态资源移动到基础镜像，构建镜像。

由于 Vue 是需要 node 环境才能镜像打包的，所以第一步骤需要一个 node 环境，之前我是在构建镜像的环境中安装了 node 环境的，然后进行打包，执行步骤比较简单：

```
npm install
npm run build
```

执行完上面的 npm 命令之后，就会在项目代码中生成 dist（默认目录）静态资源包，然后就可以执行 Dockerfile 镜像镜像构建，下面是我之前的 Dockerfile 文件的内容：

```
FROM nginx:latest
ARG from_dir=dist
ARG to_dir=/usr/share/nginx/html

COPY ${from_dir} ${to_dir}
```

其实这个目录也就是一个文件夹的移动操作，非常简单，但是 dist 目录是需要提前镜像打包产生的。

可以看到，上面的两个步骤，使用了不同的环境，打包静态资源的时候是在 node 环境，而最终的运行环境是 nginx 基础镜像中，所以这很符合多阶段构建的场景。

### 多阶段构建步骤

Dockerfile 中的 multi-stage 特性允许在一个 Dockerfile 引用多个基础镜像，可以对每个引用的镜像进行单独的操作，然后可以将每个镜像中的文件等内容进行传递。

直接来看 Dockerfile 文件：

```
FROM node:latest AS stage
WORKDIR /opt/build
COPY . .
RUN npm config set registry https://registry.npm.taobao.org/ && \
    npm install && \
    npm audit fix && \
    npm run build

FROM nginx:latest
COPY --from=stage /opt/build/dist /usr/share/nginx/html
```

可以看到这个 Dockerfile 是有用两个 FROM 命令的，第一个从 node 基础镜像进行构建，执行的步骤就是 npm 的打包，执行完成之后就会在镜像中生成 dist 资源文件夹了，这个时候就开始从 nginx 基础镜像进行构建，需要执行的目录就是从第一个镜像中把生成的目录复制过来即可。

这里就涉及到两个“语法糖”，第一个是 `FROM baseimage AS xxx` 这个很好理解，就是将这个构建步骤打个标记，用 xxx 来代表，后面可以用这个名词来表示这个阶段，第二个是 `COPY --from=xxx` 这个 `--from` 参数表示的就是从某个构建阶段的镜像中复制，而不是从本地，这个也即是多阶段构建的精髓所在，就是镜像之前可以传递文件。

### 构建结果

查看一下多阶段构建和单阶段构建出的镜像的大小：

```
[alex@CentOS-2 ~]$ docker images
REPOSITORY                                         TAG                 IMAGE ID            CREATED             SIZE
registry.cn-shenzhen.aliyuncs.com/tendcode/hao     latest              9f7e793c7d5e        3 hours ago         110MB
hao                                                test                f5ceb0cf0f52        29 hours ago        110MB
```

**总结**：可以看到，仅仅一个 Dockerfile 就可以完成从项目代码到最终形成镜像的过程，可谓是非常方便，而且非常的直观，可以让人清楚的知道在项目从代码到镜像的过程中到底经历了些什么。