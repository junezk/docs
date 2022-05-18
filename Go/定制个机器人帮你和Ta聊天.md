# [定制个机器人帮你和Ta聊天](https://segmentfault.com/a/1190000040777294)

## 项目说明

`chatbot` 是一个通过已知对话数据集快速生成回答的 Go 问答引擎。

### 为啥会有 `chatbot` 项目呢？

好多年前，当我们需要一个聊天机器人的时候，我是先用了 `ChatterBot`，但是使用下来，我们的1.2亿对话语料训练后的模型回答一个问题需要21秒左右，实在没法接受。仔细看了 `ChatterBot` 源码之后，我用 Go 重新实现了一个，并用 go-zero 的 MapReduce 框架做了并行优化，结果我们一个回答平均耗时大概18毫秒。

国庆假期，我有点空闲时间，所以就把这个项目整理了开源出来，一是给大家一个实际的 go-zero 的 MapReduce 示例；二是也提供大家一个闲聊机器人的项目玩玩。

BTW：后续我可能会开源智能客服机器人的项目，可以关注我的github:

[https://github.com/kevwan](https://link.segmentfault.com/?enc=zSOfOqHhG7TKaBSAPE35Jw%3D%3D.vnTclbr%2FSORiMbHpfzUAVwBl8cE0UXOlXComl5vtac0%3D)

### 代码目录和命令行使用说明

#### bot

问答引擎，可以自定义自己的匹配算法

#### cli

- train

  训练给定的问答数据并生成 `.gob` 文件

  - `-d` 读取指定目录下所有 `json` 和 `yaml` 语料文件
  - `-i` 读取指定的 `json` 或 `yaml` 语料文件，多个文件用逗号分割
  - `-o` 指定输出的 `.gob` 文件
  - `-m` 定时打印内存使用情况

- ask

  一个示例的问答命令行工具

  - `-v` verbose
  - `-c` 训练好的 `.gob` 文件
  - `-t` 数据几个可能的答案

## 数据格式

如果你有语料数据，可以自行整理用来训练。

数据格式可以通过 `yaml` 或者 `json` 文件提供，参考 `https://github.com/kevwan/chatterbot-corpus` 里的格式。大致如下：

```yaml
categories:
- AI
conversations:
- - 什么是ai
  - 人工智能是工程和科学的分支,致力于构建具有思维的机器。
- - 你是什么语言编写的
  - Python
- - 你听起来像机器
  - 是的,我受到造物者的启发
- - 你是一个人工智能
  - 那是我的名字。
```

## 致谢

go-zero - [https://github.com/zeromicro/...](https://link.segmentfault.com/?enc=eGCF2zjGy77fxI5DGztxvw%3D%3D.TPoEFf5QgjsVKI0QaHFBqrrmKzd9vWmI9bEioJaHM0tar%2FmO3NeHLc883ZpmWyuu)

`go-zero` 的 `core/mr` 包的 `MapReduce` 实现使 `chatbot` 的回答效率得到了巨大的提升！

ChatterBot - [https://github.com/gunthercox...](https://link.segmentfault.com/?enc=9If1XS1RYqmTyKgmxe5kTQ%3D%3D.MnGWDnOLfL2B5fDPGqY4n0C9Ozq0dvYdqXrbCQixWsrIpWIttOtqAl%2Fdn2QUVlUn)

最早我是使用 [ChatterBot](https://link.segmentfault.com/?enc=fVSvKJ1hSyDO87932pXWKQ%3D%3D.6INa8vDgxJP37jpBWDVYKQbA75h29QitSYVESIYKuZXCb9r4xTw%2BCvFyJDYemfxX) 的，但由于回答太慢，所有后来只能自己实现了，感谢 [ChatterBot](https://link.segmentfault.com/?enc=g4BAfRDqdvkB%2Bb%2F6iQLMqA%3D%3D.3V6JGOIOsQRODAuUzjUe%2BlMJvODwOJMWXJkKRO4PdB5eCEW6fYhJ8VCjXrpAo36Y)，非常棒的项目！

## 项目地址

[https://github.com/kevwan/chatbot](https://link.segmentfault.com/?enc=0P515cXInHBhnUvarkDzsg%3D%3D.NGOH1KdzZ1udRqu%2B8BioZoPU1z4Loi5KEA5csKs6Zpia3Zhy44tQxL2FUssD1F1z)

欢迎使用并 **star** 支持！