# Docker最全教程之使用 Visual Studio Code玩转Docker（二十一）

## **前言**

VS Code是一个年轻的编辑器，但是确实是非常犀利。通过本篇，老司机带你使用VS Code玩转Docker——相信阅读本篇之后，无论是初学者还是老手，都可以非常方便的玩转Docker了！所谓是“工欲善其事必先利其器”，VS Code，你值得拥有！

## **使用 Visual Studio Code玩转Docker**

Visual Studio是我们熟知的宇宙第一IDE，而Visual Studio Code（简称VS Code）则是微软推出的开源的跨平台编辑器，自从出世，一直是战斗力爆表——短短4年，就已拔得头筹，并且得到了众多开发者的拥护。如下图所示，以下是Stack Overflow 的 2018 年开发者最受欢迎的开发工具调查结果：

![Docker最全教程之使用 Visual Studio Code玩转Docker（二十一）](Docker最全教程之使用 Visual Studio Code玩转Docker（二十一）.assets/b324f7ae664b4c13978ffb2499c08076.jpg)

在Stack Overflow 的 2018 年开发者调查中，VSCode 成为了最受欢迎的开发工具

目前VisualStudio Code已经拥有了超过一万个插件，插件市场生态是极其丰富。同时其对所有的编程语言都非常友好（体验很不错），包括Docker。接下来，我们就说说Visual Studio Code对Docker的一些支持。

## **官方扩展插件Docker**

VS Code提供了对Docker支持的一些官方扩展，我们可以按Ctrl + Shift + X打开“扩展”视图，然后搜索docker以过滤结果，最后选择Microsoft Docker扩展进行安装：

![Docker最全教程之使用 Visual Studio Code玩转Docker（二十一）](Docker最全教程之使用 Visual Studio Code玩转Docker（二十一）.assets/b6429219c2b24273b5b0922d6a71a196.jpg)



使用此Docker扩展可以非常方便的从VisualStudio Code构建，管理和部署容器化应用程序，主要体现在以下几点：

- 自动生成dockerfile、docker-compose.yml和.dockerignore文件（按F1并搜索Docker：将Docker文件添加到Workspace）；

![Docker最全教程之使用 Visual Studio Code玩转Docker（二十一）](Docker最全教程之使用 Visual Studio Code玩转Docker（二十一）.assets/d11f630882c44e9e9a9520e792c35c9c.jpg)

- 语法突出高亮显示以及docker-compose.yml和Dockerfile文件的智能提示

![Docker最全教程之使用 Visual Studio Code玩转Docker（二十一）](Docker最全教程之使用 Visual Studio Code玩转Docker（二十一）.assets/dc9e4d1790db49fca591a4451a8b3f41-1560652082207.jpg)

![Docker最全教程之使用 Visual Studio Code玩转Docker（二十一）](Docker最全教程之使用 Visual Studio Code玩转Docker（二十一）.assets/dc532e7edd724f50868968e5eb23983a.jpg)

- 悬停提示；

![Docker最全教程之使用 Visual Studio Code玩转Docker（二十一）](Docker最全教程之使用 Visual Studio Code玩转Docker（二十一）.assets/0e7d59a98c834626ac394d633e53d260.jpg)

- Dockerfile文件的语法检查和分析，会提示警告或错误；

![Docker最全教程之使用 Visual Studio Code玩转Docker（二十一）](Docker最全教程之使用 Visual Studio Code玩转Docker（二十一）.assets/44481c033b0b46d9a62c82e5d0f721e6.jpg)

- 镜像搜索和智能提示；

![Docker最全教程之使用 Visual Studio Code玩转Docker（二十一）](http://p3.pstatp.com/large/pgc-image/4480815e2b374b5db6b4cf1552947c4e)

- 集成最常见的Docker命令（例如docker build，docker push等，需按F1唤起）；

![Docker最全教程之使用 Visual Studio Code玩转Docker（二十一）](http://p9.pstatp.com/large/pgc-image/b57373ad5f2a4a588ed893b8482fb461)

- Docker镜像、容器管理；

![Docker最全教程之使用 Visual Studio Code玩转Docker（二十一）](http://p3.pstatp.com/large/pgc-image/e6b1ea9dd0f24f908861d962f12f802b)

![Docker最全教程之使用 Visual Studio Code玩转Docker（二十一）](http://p3.pstatp.com/large/pgc-image/1f9147b660544917a12cae261686b7b3)

![Docker最全教程之使用 Visual Studio Code玩转Docker（二十一）](http://p9.pstatp.com/large/pgc-image/99e2d21bdd5b4391949533e8fffc55a2)

![Docker最全教程之使用 Visual Studio Code玩转Docker（二十一）](http://p1.pstatp.com/large/pgc-image/e077315181ce4ad685488a615f87c23c)

- 其他
- 对Azure的支持（这块我们就不具体介绍了）；
- .NET Core程序调试支持；
- 连接docker-machine；
- 在Linux上允许命令。

## **Docker Compose扩展插件**

我们可以按Ctrl + Shift + X打开“扩展”视图，然后搜索Docker Compose来安装此插件，扩展如下图所示：

![Docker最全教程之使用 Visual Studio Code玩转Docker（二十一）](http://p3.pstatp.com/large/pgc-image/d812d7aff9d14fef9ec7a593fb6c9bba)

该扩展支持以下功能：

- 管理Compose的工程（ Start、Stop、Up, Down）；

![Docker最全教程之使用 Visual Studio Code玩转Docker（二十一）](http://p3.pstatp.com/large/pgc-image/6e748dc2ed5f4b24ad58cdff0e9a32af)

- 管理Compose服务（支持Up, Shell, Start, Stop, Restart,Build, Kill, Down）；

![Docker最全教程之使用 Visual Studio Code玩转Docker（二十一）](http://p3.pstatp.com/large/pgc-image/ac50180ac9bb4a89bfda0eaa5152079d)

- 支持多个根；

## **最后**

VS Code是一个年轻的编辑器，但是确实是非常犀利。通过这两个插件，无论是初学者还是老手，都可以非常方便的玩转容器了！所谓是“工欲善其事必先利其器”，VS Code，你值得拥有！