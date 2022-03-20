# dpkg: error processing package XXX (--configure) 解决方法

通过执行下面的命令可以解决该问题：

```shell
$ sudo mv /var/lib/dpkg/info/ /var/lib/dpkg/info_old/
$ sudo mkdir /var/lib/dpkg/info/
$ sudo apt-get update
...
$ sudo apt-get -f install
    Reading package lists... Done
    Building dependency tree
    Reading state information... Done
    0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
    7 not fully installed or removed.
    After this operation, 0 B of additional disk space will be used.
    Setting up bluez (4.101-0ubuntu13.1) ...
    Setting up blueman (1.23-git201403102151-1ubuntu1) ...
    Setting up bluetooth (4.101-0ubuntu13.1) ...
    Setting up bluez-alsa:amd64 (4.101-0ubuntu13.1) ...
    Setting up bluez-alsa:i386 (4.101-0ubuntu13.1) ...
    Setting up bluez-gstreamer (4.101-0ubuntu13.1) ...
    Setting up bluez-utils (4.101-0ubuntu13.1) ...
$ sudo mv /var/lib/dpkg/info/* /var/lib/dpkg/info_old/
$ sudo rm -rf /var/lib/dpkg/info
$ sudo mv /var/lib/dpkg/info_old/ /var/lib/dpkg/info/
```

输入上述命令之后，在执行 `sudo apt-get update` 和 `sudo apt-get upgrade` 就不会有问题了。