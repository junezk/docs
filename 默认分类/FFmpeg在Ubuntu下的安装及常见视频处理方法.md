# FFmpeg在Ubuntu下的安装及常见视频处理方法

## 一、安装

可通过PPA进行安装

```
sudo add-apt-repository ppa:kirillshkrogalev/ffmpeg-next
sudo apt-get update
sudo apt-get install ffmpeg
```

## 二、相关概念

###比特率
比特率，是一个决定音视频总体质量的参数。他决定每个时间单位处理的bit数，英文为 bit rate，描述每秒钟输出多少 KB 的参数，单位是 Kbps，也就是 kbit/s，8Kbit/s = 1KB/s。压缩同一个视频，视频编码率越大，文件体积越大，画质越好。

MP3一般使用的比特率为 8~320kbps。

**设置比特率：**
比特率决定处理1s的编码流需要多少bits，设置用-b选项。区分音视频用-b:a和-b:v
例如：设置整体1.5Mbit每秒

```
ffmpeg -i file.avi -b 1.5M file.mp4
ffmpeg -i input.avi -b:v 1500K output.mp4
```

### 帧数

每秒钟播放的图片数，单位 fps（英文：Frames Per Second），帧率就是每秒编码进视频文件的帧数目。人类的眼睛需要每秒至少15帧才能将图像连贯在一起。帧率的单位是HZ，LCD显示一般有60Hz的平率。高的帧率可以得到更流畅、更逼真的动画。一般来说30fps就是可以接受的，但是将性能提升至60fps则可以明显提升交互感和逼真感，但是一般来说超过75fps一般就不容易察觉到有明显的流畅度提升了。如果帧率超过屏幕刷新率只会浪费图形处理的能力，因为显示器不能以这么快的速度更新，这样超过刷新率的帧率就浪费掉了。

在同一视频，同一码率的情况下，帧数越大，则画质越不好。尤其是运动的画面。因为每张画面会分担每秒有限的文件体积，如果画面越多，那么每张画面所能表现的内容就越有限。

当画面的FPS达到60帧/秒时，已经能满足绝大部分应用需求。一般情况下，如果能够保证游戏画面的平均FPS能够达到30帧/秒，那么画面已经基本流畅；能够达到50帧/秒，就基本可以体会到行云流水的感觉了。一般人很难分辨出60 帧/秒与100帧/秒有什么不同。

**帧率设置**

使用-r选项

```
ffmpeg -i input.avi -r 30 output.mp4
```

### 分辨率

最好理解的概念了，表示画面的大小，单位是像素 px。

和编码率的关系：越高的分辨率，需要越高的编码率，因为图像的细节多了，需要的文件体积也应该增大，否则还不如画面小一些，你会发现同一码率，画面越大，图像的马赛克程度越明显。

### 采样率

每秒钟对音频信号的采样次数，采样频率越高声音还原度越高，声音更加自然。单位是赫兹 Hz。音频文件一般使用的采样率是 44100 Hz ，也就是一秒钟采样 44100 次，之所以使用这个数值是因为经过了反复实验，人们发现这个采样精度最合适，低于这个值就会有较明显的损失，而高于这个值人的耳朵已经很难分辨，而且增大了数字音频所占用的空间。我们所使用的CD的采样标准就是44.1k，目前44.1k还是一个最通行的标准。

## 三、常见用法

### 主要参数：

- -i 设定输入流

- -f 设定输出格式

- -ss 开始时间

### 视频参数：

- -b 设定视频流量，默认为200Kbit/s

- -r 设定帧速率，默认为25

- -s 设定画面的宽与高

- -aspect 设定画面的比例

- -vn 不处理视频

- -vcodec 设定视频编解码器，未设定时则使用与输入流相同的编解码器

### 音频参数：

- -ar 设定采样率

- -ac 设定声音的Channel数

- -acodec 设定声音编解码器，未设定时则使用与输入流相同的编解码器

- -an 不处理音频


### 用法举例

显示视频信息

```
ffmpeg -i input.avi
```

1. 格式转换

   ffmpeg最常用功能就是格式转换，在这里要特别提的是，音、视频文件格式有两个容器格式（如mov、flv)与编码格式（如H.264）

   ```
   ffmpeg -i input.flv output.mp4
   ```

2. 尺寸变换

   ```
   ffmpeg -i input.mp4 -s 640x360 output.mp4
   ```

   

3. 剪切视频段

   ```
   ffmpeg -i input.mp4 -ss 5 -t 10 output.mp4
   ```

   上面的命令 -ss 5 指定从输入视频第5秒开始截取，-t 10 指明最多截取10秒。 但是上面的命令可能会比较慢，更好的命令如下：

   ```
   ffmpeg -ss 5 -i input.mp4 -t 10 -c:v copy -c:a copy output.mp4
   ```

   上面的命令把 -ss 5 放到 -i 前面，与原来的区别是，这样会先跳转到第5秒在开始解码输入视频，而原来的会从开始解码，只是丢弃掉前5秒的结果。
    而 -c:v copy -c:a copy 标示视频与音频的编码不发生改变，而是直接复制，这样会大大提升速度，因为这样就不需要完全解码视频（视频剪切也不需要完全解码）

   ***注意：-vcodec 有一个缩写叫做  -c:v  ，-acodec 有一个缩写叫做 -c:a 。***

4. 改变FPS

   FFmpeg可以用于降低或提高视频的帧率，因为信息丢失不可逆法则，提高帧率只会简单地让某些帧的画面多重复一次或多次，所以提高帧率不会提高画质。

   ```
   ffmpeg -i input.mp4 -r 30 output.mp4
   ```

   上面的命令，不论原始视频帧率是多少，输出视频都会是30帧每秒。这种情况之下视频的时间轴不会变化，不会有慢动作或快动作的效果。

5. 截取图片

   视频10秒的地方(-ss 参数)截取一张1920x1080尺寸大小的，格式为jpg的图片 -ss后跟的时间单位为秒

   ```
   ffmpeg -i input_video.mp4 -y -f image2 -t 0.001 -ss 10 -s 1920x1080 output.jpg
   ```

   把视频的前30帧转换成一个Gif

   ```
   ffmpeg -i input_video.mp4 -vframes 30 -y -f gif output.gif
   ```

   将视频转成 gif

   ```
   ffmpeg -ss 00:00:00.000 -i input.mp4 -pix_fmt rgb24 -r 10 -s 320x240 -t 00:00:10.000 output.gif
   ```

   将输入的文件从(-ss)设定的时间开始以10帧频率，输出到320x240大小的 gif 中，时间长度为-t 设定的参数。通过这样转换出来的 gif 一般都比较大，可以使用 [ImageMagick](http://www.imagemagick.org/) 来优化图片的大小。

## 四、转码时输出信息

```
frame=   28 fps=0.0 q=0.0 size=       2kB time=00:00:01.49 bitrate=  11.3kbits/s
frame=   30 fps= 17 q=-0.0 size=      13kB time=00:00:01.49 bitrate=  71.1kbits/
frame=   34 fps= 15 q=-0.0 size=      20kB time=00:00:01.66 bitrate=  99.9kbits/
frame=   38 fps= 13 q=-0.0 size=      31kB time=00:00:01.83 bitrate= 138.1kbits/
frame=   42 fps= 12 q=-0.0 size=      40kB time=00:00:02.00 bitrate= 165.1kbits/
frame=   46 fps= 11 q=-0.0 size=      49kB time=00:00:02.17 bitrate= 185.4kbits/
frame=   50 fps= 10 q=-0.0 size=      57kB time=00:00:02.34 bitrate= 199.3kbits/
frame=   54 fps= 10 q=-0.0 size=      63kB time=00:00:02.51 bitrate= 204.9kbits/
frame=   58 fps=9.5 q=-0.0 size=      74kB time=00:00:02.68 bitrate= 226.2kbits/
frame=   62 fps=9.2 q=-0.0 size=      85kB time=00:00:02.68 bitrate= 260.5kbits/
frame=   65 fps=8.8 q=-0.0 size=      92kB time=00:00:02.85 bitrate= 264.9kbits/
```



FFmpeg 确实不会显示进度条和百分比，不过，它会给你比进度条和百分比还要多的信息。

1.  最左边的 `frame= 65` 是转码所进行到的帧数，显示 65 就表示现在已经转到了第 65 帧。
2.  第二个 `fps=8.8` 中的 FPS 就是 Frame per Second ，也就是现在电脑每秒所处理的帧的数量。注意这个数字跟视频的帧率并无关系。
3.  其实我也不知道后面那个 `q=-0.0` 是什么意思。
4.  接下来的 `size= 92kB` 表示现在已经转换出来的视频的体积，这个数字只会越变越大啊。
5.  第五个 `time=00:00:02.85` 顾名思义就是时间了，它是已经转换出来的视频的时间。在我看来，它也是一个比百分比进度条更加精准的进度显示。

官方文档：https://www.ffmpeg.org/ffmpeg.html#Video-Options