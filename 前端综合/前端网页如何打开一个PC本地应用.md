# 前端网页如何打开一个PC本地应用

设想一个场景，当我们在浏览一个网页并且需要下载某个资源时，你的电脑可能经常会跳出一个提示框，询问你是否需要打开“迅雷”。当我们点击“是”，则会唤醒该本地应用进行下载任务。

针对这个场景产生了一个疑问，网页是如何打开PC端应用的呢？

本文针对`Windows`系统和`MacOS`系统进行讨论。

# 自定义协议

在薄荷FE的日常开发中，因为需要与app频繁交互，app开发人员定义了相关协议：`boohee://`，通过该协议，我们可以唤起薄荷app。

通过这个场景作者衍生出一个想法，PC端的应用是否也可以通过类似的协议被打开呢？

# Windows

## 注册表

注册表是Microsoft Windows中的一个重要的数据库，用于存储系统和应用程序的设置信息。

它是Windows操作系统中的一个核心数据库，其中存放着各种参数，可以直接控制一些Windows应用程序的运行。

在Windows环境中，我们可以通过注册表来定义打开软件的协议。

### 如何查看注册表中的协议?

Windows系统中自带了注册表编辑器，通过Windows+r打开运行，输入"regedit"，打开注册表编辑器。

![注册表编辑器](前端网页如何打开一个PC本地应用.assets/16e4404b8bea4621)

我们需要的有关打开应用的注册表配置就存在`HEY_CLASSES_ROOT`下。

### HEY_CLASSES_ROOT

`HKEY_CLASSES_ROOT`根键中主要包含的是所有启动应用程序需要的信息，其中包括：

1. 所有扩展名及应用程序和文档之间的关联信息。
2. 所有驱动程序的名字。
3. 当作指针的字符串，指向它们代表的实际文件。
4. 类标识CLSID，这点在访问子健信息的时候非常重要，因为Windows中访问了子健的信息都是用CLSID来代替的。这里的标识在Windows XP系统中是唯一的。
5. DDE和OLE信息。对于每个文件关联都可以使用DDE和OLE功能。
6. 应用程序和文档使用的图标

## 示例： 打开postman

![postman](前端网页如何打开一个PC本地应用.assets/16e4404b8c06c7f2)

![导出](前端网页如何打开一个PC本地应用.assets/16e4404b8bfe172d)

点击postman文件夹，可以看到右侧有个默认属性定义了`URL:postman`，导出该注册表可以看到如下配置：

```
    Windows Registry Editor Version 5.00
    
    [HKEY_CURRENT_USER\Software\Classes\postman]
    "URL Protocol"=""
    @="URL:postman"
    
    [HKEY_CURRENT_USER\Software\Classes\postman\shell]
    
    [HKEY_CURRENT_USER\Software\Classes\postman\shell\open]
    
    [HKEY_CURRENT_USER\Software\Classes\postman\shell\open\command]
    @="\"C:\\Users\\X\\AppData\\Local\\Postman\\app-6.0.10\\Postman.exe\" \"%1\""
```

`[HKEY_CURRENT_USER\Software\Classes\postman]`中的postman就是协议的名字，该配置主要通过`[HKEY_CURRENT_USER\Software\Classes\postman\shell\open\command]`中定义的地址来找到软件并启动。启动软件主要依赖以下两个配置：

```
    Windows Registry Editor Version 5.00
    
    [HKEY_CURRENT_USER\Software\Classes\postman]
    "URL Protocol"=""
    @="URL:postman"
    
    [HKEY_CURRENT_USER\Software\Classes\postman\shell\open\command]
    @="\"C:\\Users\\X\\AppData\\Local\\Postman\\app-6.0.10\\Postman.exe\" \"%1\""
```

分析一下上述的配置是什么意思：

- HKEY_CURRENT_USER\Software\Classes\postman: 定义了驱动函数的名字
- HKEY_CURRENT_USER\Software\Classes\postman\shell\open\command：定义了程序所在的路径

根据这两个配置，前端网页可以通过`postman://`协议来打开本地的postman应用。

效果展示：

 ![打开postman](前端网页如何打开一个PC本地应用.assets/16e4404b8c1a3776) 

# MacOS

在MacOS中打开应用和在Ios中相同，可以给自己的app添加`URL Schemes`。

## Info.plist

每次新建一个项目工程，Xcode都会自动创建一个Info.plist文件，这个文件的主要作用就是提供应用在运行期的一些配置。

`Info.plist`文件位于应用程序的`Contents/`子目录下，这个文件保存了应用包的元数据信息。这个文件是必备的，操作系统通过这个文件判定依赖关系和其他属性。

CFBundleURLTypes：这个应用包关联的URL。这是一个字典，指定了这个包处理的`URL schemes`以及处理方式。

以Foxmail为例，这是该应用相关的配置：

```
	<key>CFBundleURLTypes</key>
	<array>
		<dict>
			<key>CFBundleURLName</key>
			<string>mailto</string>
			<key>CFBundleURLSchemes</key>
			<array>
				<string>mailto</string>
			</array>
		</dict>
	</array>
```

其中定义了该应用包下定义的`URL Schemes`为`mailto://`。

### URL Schemes

参考链接：[什么是URL Schemes](https://sspai.com/post/31500#02)

URL Schemes 有两个单词：

- URL，我们都很清楚，`http://www.apple.com` 就是个 URL，我们也叫它链接或网址；
- Schemes，表示的是一个 URL 中的一个位置——最初始的位置，即 `://`之前的那段字符。比如`http://www.apple.com` 这个网址的 `Schemes` 是 `http`。

根据我们上面对 URL Schemes 的使用，我们可以很轻易地理解，在以本地应用为主的 iOS 上，我们可以像定位一个网页一样，用一种特殊的 URL 来定位一个应用甚至应用里某个具体的功能。而定位这个应用的，就应该这个应用的 URL 的 Schemes 部分，也就是开头儿那部分。比如短信，就是`sms`:

你可以完全按照理解一个网页的 URL ——也就是它的网址——的方式来理解一个 iOS 应用的 URL，拿苹果的网站和 iOS 上的微信来做个简单对比：

|                   |                         网页（苹果）                         |        iOS 应用（微信         |
| :---------------: | :----------------------------------------------------------: | :---------------------------: |
| 网站首页/打开应用 |            [www.apple.com](http://www.apple.com/)            |           weixin://           |
|  子页面/具体功能  | [www.apple.com/mac/（Mac页面）](http://www.apple.com/mac/（Mac页面）) | weixin://dl/moments（朋友圈） |

在这里，`http://www.apple.com` 和 `weixin://` 都声明了这是谁的地盘。然后在 `http://www.apple.com` 后面加上 `/mac` 就跳转到从属于 `http://www.apple.com` 的一个网页（Mac 页）上；同样，在 `weixin://` 后面加上 `dl/moments` 就进入了微信的一个具体的功能——朋友圈。

### 如何查询app的URL Schemes

打开finder => 应用程序 => 右击应用并选择显示包内容 => 选择`Contents`文件夹下的`Info.plist`文件

![显示包内容](前端网页如何打开一个PC本地应用.assets/16e4404b8c35115e)

![Info.plist](前端网页如何打开一个PC本地应用.assets/16e4404b8c29f214)

![URL Schemes](前端网页如何打开一个PC本地应用.assets/16e4404bcf83a0b0)

## 示例：打开QQ

打开Info.plist，查询`CFBundleURLSchemes`字段，该字段下定义的内容都可以作为`URL Schemes`打开应用。

以QQ为例，`qq://`和`tencent://`都可以触发打开本地应用。

![mac打开qq](前端网页如何打开一个PC本地应用.assets/16e4404bcf9128b1)

![openqq1](前端网页如何打开一个PC本地应用.assets/16e4404bd7655475)

![openqq2](前端网页如何打开一个PC本地应用.assets/16e4404be03e2e42)

# 实例：打开一个本地应用程序

## Windows

以网易云音乐为例，根据上述方法，我们打开注册表编辑器，去查询有没有定义打开网易云音乐的协议，发现相关注册表如下，并没有定义打开该应用的协议：

![nocloud](前端网页如何打开一个PC本地应用.assets/16e4404be07b2748)

好在我们在Windows可以自定义一个注册表并使之生效，我们可以在任何一个位置创建一个`**.reg`文件，使用记事本进行编辑：

```
    Windows Registry Editor Version 5.00
    
    [HKEY_CLASSES_ROOT\cloudmusic]
    "URL Protocol"=""
    @=""
    
    [HKEY_CLASSES_ROOT\cloudmusic\shell\open\command]
    @="\"D:\\CloudMusic\\cloudmusic.exe\"\"%1\""
```

在此实例中作者将其命名为`cloudmusic.reg`。

注意：

1. 文件命名与协议名称无关，只与`[HKEY_CLASSES_ROOT]`后定义的字段有关；
2. `@="\"D:\\CloudMusic\\cloudmusic.exe\"\"%1\""`中的"%1"为参数，不能随意删除。

保存reg文件之后，双击文件将其中的配置注册到系统中：

![cloudreg1](前端网页如何打开一个PC本地应用.assets/16e4404be078e8a6)

![cloudreg2](前端网页如何打开一个PC本地应用.assets/16e4404c1302737c)

![cloudreg3](前端网页如何打开一个PC本地应用.assets/16e4404c1b34a6c5)

添加注册表配置之后，我们再次进入注册表编辑器中查看，发现相关配置已展示在列表中：

![hascloud](前端网页如何打开一个PC本地应用.assets/16e4404c2465e559)

来验证一下是否能打开一个应用：

![打开网易云音乐](前端网页如何打开一个PC本地应用.assets/16e4404c24828992)

配置成功！

### 如何查找应用所在地址

打开菜单找到对应的应用，单击右键选择“打开文件位置”：

![menu](前端网页如何打开一个PC本地应用.assets/16e4404c249ec7b2)

找到文件之后单击右键选择“属性”，查看所在系统位置：

![属性](前端网页如何打开一个PC本地应用.assets/16e4404c352951d0)

属性中的“目标”就是软件所在的系统位置，复制到注册表中即可：

![地址](前端网页如何打开一个PC本地应用.assets/16e4404c5728fc89)

## MacOS

在Windows系统中，我们可以通过注册表来定义打开应用的协议，那么在MacOS中是否也可以通过修改`Info.plist`来进行配置呢？

以Teambition为例，我们将以下配置加入它的Info.plist文件中：

```
    <key>CFBundleURLTypes</key>
    <array>
    	<dict>
    		<key>CFBundleURLName</key>
    		<string>teambition</string>
    		<key>CFBundleURLSchemes</key>
    		<array>
    			<string>teambition</string>
    		</array>
    	</dict>
    </array>
```

重新运行该应用，报错提示如下：

![tb报错](前端网页如何打开一个PC本地应用.assets/16e4404c5f877af0)

在safari中输入`teambition://`尝试打开，提示如下：

![tb报错2](前端网页如何打开一个PC本地应用.assets/16e4404c67da6c41)

阿欧，配置失败了，我们没有权限去修改MacOS上的应用。

# 如何判断自定义协议是否存在

参考指路：[使用JS检测自定义协议是否存在](https://www.cnblogs.com/tangjiao/p/9646855.html)

# 总结

当前端页面需要打开本地应用时，需要先查询本地应用是否有配置协议，如果未配置，在Windows环境下可以通过注册表进行添加，MacOS环境则无法添加。每个使用者都需要在自己的电脑中进行注册表的配置，局限性较大。

参考文档：

[MacOS 开发 - 使用 safari 打开Mac应用](https://blog.csdn.net/lovechris00/article/details/79689598)

[MacOS 给自己的 app 添加 URL Scheme](https://blog.csdn.net/lovechris00/article/details/77896410)