# Python新利器之pipenv

[TOC]

## 前言

之前学习异步`asyncio`库的时候，因为`asyncio`库支持Python3.5以上的版本，而我的Ubuntu14.04只有Python3.4，虽然下载了Python3.6，但是想直接利用ipython3或者pip3调用Python3.6相关的东西有点困难，可能是我手法不对，有点混乱。

之前只是简单的用过`virtualenv`，直到发现了这个`pipenv`，有点吊炸天。

> Python开发者应该听过pip、easy_install和virtualenv，如果看过我的书应该还知道 virtualenvwrapper、virtualenv-burrito和autoenv，再加上pyvenv、venv（Python 3标准库）、pyenv…
> 额，是不是有种发懵的感觉？
> 那么现在有个好消息，你可以只使用终极方案: pipenv + autoenv（可选）。

## pipenv 都包含什么？

pipenv 是 Pipfile 主要倡导者、requests 作者 Kenneth Reitz 写的一个命令行工具，主要包含了Pipfile、pip、click、requests和virtualenv。Pipfile和pipenv本来都是Kenneth Reitz的个人项目，后来贡献给了pypa组织。Pipfile是社区拟定的依赖管理文件，用于替代过于简陋的 requirements.txt 文件。

Pipfile的基本理念是：

Pipfile 文件是 TOML 格式而不是 requirements.txt 这样的纯文本。
一个项目对应一个 Pipfile，支持开发环境与正式环境区分。默认提供 default 和 development 区分。
提供版本锁支持，存为 Pipfile.lock。
click是Flask作者 Armin Ronacher 写的命令行库，现在Flask已经集成了它。

接下来，我们看看怎么使用它吧

## 安装

```
$ pip install pipenv

```

## 用法

在使用`pipenv`之前，必须彻底的忘记`pip`这个东西

新建一个准备当环境的文件夹pipenvtest，并cd进入该文件夹：
`pipenv --three` 会使用当前系统的Python3创建环境

`pipenv --python 3.6` 指定某一Python版本创建环境

`pipenv shell` 激活虚拟环境

`pipenv --where` 显示目录信息
`/home/jiahuan/pipenvtest`

`pipenv --venv` 显示虚拟环境信息
`/home/jiahuan/.local/share/virtualenvs/pipenvtest-9KKRH3OW`

`pipenv --py` 显示Python解释器信息
`/home/jiahuan/.local/share/virtualenvs/pipenvtest-9KKRH3OW/bin/python`

`pipenv install requests` 安装相关模块并加入到Pipfile

`pipenv install django==1.11` 安装固定版本模块并加入到Pipfile

`pipenv graph` 查看目前安装的库及其依赖

```
requests==2.18.4
  - certifi [required: >=2017.4.17, installed: 2017.11.5]
  - chardet [required: <3.1.0,>=3.0.2, installed: 3.0.4]
  - idna [required: >=2.5,<2.7, installed: 2.6]
  - urllib3 [required: >=1.21.1,<1.23, installed: 1.22]

```

`pipenv check`检查安全漏洞

```
Checking PEP 508 requirements…
Passed!
Checking installed package safety…
All good! 

```

`pipenv uninstall --all` 卸载全部包并从Pipfile中移除

```
Found 5 installed package(s), purging…
Uninstalling certifi-2017.11.5:
  Successfully uninstalled certifi-2017.11.5
Uninstalling chardet-3.0.4:
  Successfully uninstalled chardet-3.0.4
Uninstalling idna-2.6:
  Successfully uninstalled idna-2.6
Uninstalling requests-2.18.4:
  Successfully uninstalled requests-2.18.4
Uninstalling urllib3-1.22:
  Successfully uninstalled urllib3-1.22

```

跟上面graph命令显示的内容对应

## 出现个报错

之后随意测试的时候 使用`pipenv --two` 想创建一个基于Python2.7的虚拟环境时出了点问题。报了这样一个错误
`TypeError: 'NoneType' object is not subscriptable`
而使用`pipenv --python 3.6`却没有问题（自带的是Python3.5，Python3.6新安装的，这让我很纳闷，明天去公司试一试。

经过测试:在公司ubuntu机器上可以使用 ,今晚再回家里试试...
.....
回家仔细观察了报错，原来是我pip源的配置文件出了点错，多了个空格，囧 ～

## 更换国内源

pipenv install 安装模块时有时候会很慢
可以设置国内源：`Pipfile`文件中`[source]`下面`url`属性，比如修改成：`url = "https://pypi.tuna.tsinghua.edu.cn/simple"`

## 小结

这里写了一个pipenv常用的命令，很不错的工具，pip与virtualenv的结合体，值得一用。