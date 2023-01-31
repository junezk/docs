# [MAC下环境变量配置文件及问题](https://blog.csdn.net/axiaofeijia/article/details/124900498#MAC_0)

## 一、配置文件的加载

### 1、全局配置

| 配置文件      | 描述              |
| ------------- | ----------------- |
| /etc/profile  | 系统级别          |
| /etc/paths    | 系统级别          |
| /etc/bashrc   | 基于bash（shell） |
| /etc/zprofile | 基于zsh（shell）  |
| /etc/zshrc    | 基于（shell）     |

### 2、用户配置

| 配置文件       | 描述              |
| -------------- | ----------------- |
| ~/bash_profile | 基于bash（shell） |
| ~/etc/zprofile | 基于zsh（shell）  |
| ~/etc/zshrc    | 基于（shell）     |

> 1、全局文件比用户配置文件优先级高！！
> 2、注意查看使用的shell类型 zsh？bash？ 到对应的配置文件去配置

## 二、环境变量

```bash
#BASH环境变量设置

#Java
export JAVA_HOME="/Library/Java/JavaVirtualMachines/jdk1.8.0_221.jdk/Contents/Home"
PATH="$PATH:$JAVA_HOME/bin"
#Java End

# HomeBrew
export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles
export PATH="/usr/local/bin:$PATH"
export PATH="/usr/local/sbin:$PATH"
# HomeBrew End
#Mysql
export PATH="/Library/Mysql/bin:$PATH"
#Mysql End

#Tomcat
export TOMCAT_HOME="/Library/Tomcat/apache-tomcat-xxx/"
export Tomcat="/Library/Tomcat/apache-tomcat-xxx/bin"
#Tomcat End


```

> 1、设置XX_HOME也方便查找文件存放位置
> 2、规范书写，如 #Tomcat 、#Tomcat End标识
> 3、export 命令用于设置或显示环境变量, export[变量名]=[变量值],在终端通过 eho命令输出显示变量，eg：echo $TOMCAT_HOME
> 4、PATH设置时，要加‘：PATH’，来表示之前PATH的值 ，不加入使得PATH为设置的值（唯一）。注：冒号表示并列。
> 5、Mac自带jdk 如果不设置java环境变量，通过java -version也会出现java版本