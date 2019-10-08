# 用Python帮你实现IP子网计算

# 0. 前言

IP地址目前存在两个版本：IPv4和IPv6，平常我们见到最多的就是IPv4了，如`192.168.1.1/24`，当然,IPv4地址池资源紧缺，IPv6已悄然大量部署了。

我们在设计网络架构时必须要对设备互联地址、环回地址、业务地址进行规划，那怎么规划？给你一个A类地址你怎么办？最重要是不是得计算？口算怕不准确吧？心算行不行，就不怕你没这本事，哈哈！

下面请用python帮你搞定这一切吧！

# 1. ipaddress模块介绍

## 1.1 IP主机地址

**说明：不带掩码**

怎么判断是ipv4地址，还是ipv6地址呢？使用`ipaddress.ip_address()` 函数可以来知晓:

```
>>> ipaddress.ip_address('192.168.1.1')
IPv4Address('192.168.1.1')
>>> ipaddress.ip_address('192.168.1.1').version
4

>>> ipaddress.ip_address('fe80::1')
IPv6Address('fe80::1')
>>> ipaddress.ip_address('fe80::1').version
6
```

如果带上掩码就会报错：

```
>>> ipaddress.ip_address('192.168.1.1/32')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/lib/python3.5/ipaddress.py", line 54, in ip_address
    address)
ValueError: '192.168.1.1/32' does not appear to be an IPv4 or IPv6 address
```

## 1.2 定义网络

**说明：表示网段**

一个IP地址，通常由网络号+网络前缀组成，如`192.168.1.0/24`,可以通过`ipaddress.ip_network`函数来表示，缺省情况下，python只能识别网络号，如果是IP主机就会报错，当然你可以通过`strict=False`来避免。

```
>>> ipaddress.ip_network('192.168.1.0/24')
IPv4Network('192.168.1.0/24')

#缺省，输入主机位就会报错
>>> ipaddress.ip_network('192.168.1.1/24')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/lib/python3.5/ipaddress.py", line 74, in ip_network
    return IPv4Network(address, strict)
  File "/usr/lib/python3.5/ipaddress.py", line 1536, in __init__
    raise ValueError('%s has host bits set' % self)
ValueError: 192.168.1.1/24 has host bits set  #提示是主机IP

#修改位非严格模式，缺省为strict=True
>>> ipaddress.ip_network('192.168.1.1/24' , strict=False)
IPv4Network('192.168.1.0/24')   #返回网络号
```

## 1.3 主机接口

**说明：表示接口地址(ip/掩码)** 一般在路由器、交换机、防火墙接口上配置IP地址，格式如192.168.1.1/24，如果使用以上`ipaddress.ip_address()`和`ipaddress.ip_network`函数的话，就不太好表示，那么可以通过`ipaddress.ip_interface()`函数类表示。

```
>>> ipaddress.ip_interface('192.168.1.1/24')
IPv4Interface('192.168.1.1/24')
```

## 1.4 检查address/network/interface对象

### 1.4.1 检查IP版本(v4或者v6)：

```
>>> ipaddress.ip_address('192.168.1.1').version
4
>>> ipaddress.ip_address('fe80::1').version
6
```

### 1.4.2 从接口IP获取网段

```
>>> ipaddress.ip_interface('192.168.1.1/24').network
IPv4Network('192.168.1.0/24')

>>> ipaddress.ip_interface('fe80::/64').network
IPv6Network('fe80::/64')
```

### 1.4.3 计算网段有多少个IP地址

```
>>> ipaddress.ip_network('192.168.1.0/24').num_addresses
256

>>> ipaddress.ip_network('fe80::/64').num_addresses
18446744073709551616
```

### 1.4.4 计算网段有多少个可用IP地址

```
>>> net = ipaddress.ip_network('192.168.1.0/24')
>>> for x in net.hosts():
...     print(x)
... 
192.168.1.1
192.168.1.2
    ...
192.168.1.100
192.168.1.101
    ...
192.168.1.254

>>> [x for x in net.hosts()][0]     #获取第一个可用IP
IPv4Address('192.168.1.1')
>>> [x for x in net.hosts()][-1]    #获取最后一个可用IP
IPv4Address('192.168.1.254')
```

### 1.4.5 获取掩码与反掩码

```
>>> ipaddress.ip_network('192.168.1.1/24' , strict=False).netmask
IPv4Address('255.255.255.0')    #获取掩码

>>> ipaddress.ip_network('192.168.1.1/24' , strict=False).hostmask
IPv4Address('0.0.0.255')    #获取反掩码
```

## 1.6 获取网络号与广播地址

```
>>> ipaddress.ip_network('192.168.1.1/24' , strict=False).network_address
IPv4Address('192.168.1.0')      #获取网络号

>>> ipaddress.ip_network('192.168.1.1/24' , strict=False).broadcast_address
IPv4Address('192.168.1.255')    #获取广播地址
```

## 1.7 异常处理

如果遇到IP地址格式不符合要求等这些情况，那怎么处理呢？

```
#错误显示,报"ValueError"
>>> ipaddress.ip_network('192.168.1.1/24')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/lib/python3.5/ipaddress.py", line 74, in ip_network
    return IPv4Network(address, strict)
  File "/usr/lib/python3.5/ipaddress.py", line 1536, in __init__
    raise ValueError('%s has host bits set' % self)
ValueError: 192.168.1.1/24 has host bits set

#通过try-except语句来处理异常情况
>>> import ipaddress
>>> def cal_ip(net):
...     try:
...         net = ipaddress.ip_network(net)
...         print(net)
...     except ValueError:
...         print('您输入格式有误，请检查！')
... 
>>> cal_ip(net = '192.168.1.1/24')
您输入格式有误，请检查！
```

# 2. 计算IP子网代码演示

## 2.1 完整代码

```
#!/usr/bin/env python3
#-*- coding:UTF-8 -*-
#欢迎关注微信公众号：点滴技术

import ipaddress

def cal_ip(ip_net):
    try:
        net = ipaddress.ip_network(ip_net, strict=False)
        print('IP版本号： ' + str(net.version))
        print('是否是私有地址： ' + str(net.is_private))
        print('IP地址总数: ' + str(net.num_addresses))
        print('可用IP地址总数： ' + str(len([x for x in net.hosts()])))
        print('网络号： ' + str(net.network_address))
        print('起始可用IP地址： ' + str([x for x in net.hosts()][0]))
        print('最后可用IP地址： ' + str([x for x in net.hosts()][-1]))
        print('可用IP地址范围： ' + str([x for x in net.hosts()][0]) + ' ~ ' + str([x for x in net.hosts()][-1]))
        print('掩码地址： ' + str(net.netmask))
        print('反掩码地址： ' + str(net.hostmask))
        print('广播地址： ' + str(net.broadcast_address))
    except ValueError:
        print('您输入格式有误，请检查！')

if __name__ == '__main__':
    ip_net = '192.168.1.1/24'
    cal_ip(ip_net)
```

## 2.2 运行结果

```
IP版本号： 4
是否是私有地址： True
IP地址总数: 256
可用IP地址总数： 254
网络号： 192.168.1.0
起始可用IP地址： 192.168.1.1
最后可用IP地址： 192.168.1.254
可用IP地址范围： 192.168.1.1 ~ 192.168.1.254
掩码地址： 255.255.255.0
反掩码地址： 0.0.0.255
广播地址： 192.168.1.255
```

# 3. 碎碎语

怎么样，学完之后是不是很亢奋，不需要借助其他工具进行计算了吧，用python就帮你搞定了。

### 3.1 官方参考文档

[docs.python.org/3.8/howto/i…](https://link.juejin.im/?target=https%3A%2F%2Fdocs.python.org%2F3.8%2Fhowto%2Fipaddress.html)