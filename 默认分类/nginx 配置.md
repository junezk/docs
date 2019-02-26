# Nginx 配置

## Nginx虚拟目录alias和root目录

nginx是通过alias设置虚拟目录，在nginx的配置中，alias目录和root目录是有区别的：

1. alias指定的目录是准确的，即location匹配访问的path目录下的文件直接是在alias目录下查找的；

2. root指定的目录是location匹配访问的path目录的上一级目录,这个path目录一定要是真实存在root指定目录下的；

3. 使用alias标签的目录块中不能使用rewrite的break（具体原因不明）；另外，alias指定的目录后面必须要加上"/"符号！！

4. alias虚拟目录配置中，location匹配的path目录如果后面不带"/"，那么访问的url地址中这个path目录后面加不加"/"不影响访问，访问时它会自动加上"/"；

   但是如果location匹配的path目录后面加上"/"，那么访问的url地址中这个path目录必须要加上"/"，访问时它不会自动加上"/"。如果不加上"/"，访问就会失败！

5. root目录配置中，location匹配的path目录后面带不带"/"，都不会影响访问。

举例说明（比如nginx配置的域名是www.wangshibo.com）：

1. 虚拟目录和真实目录同名

    ```
    location /huan/ {
          alias /home/www/huan/;
    }
    ```

    在上面alias虚拟目录配置下，访问http://www.wangshibo.com/huan/a.html实际指定的是/home/www/huan/a.html。
    注意：alias指定的目录后面必须要加上"/"，即/home/www/huan/不能改成/home/www/huan

    上面的配置也可以改成root目录配置，如下，这样nginx就会去/home/www/huan下寻找http://www.wangshibo.com/huan的访问资源，两者配置后的访问效果是一样的！

    ```
    location /huan/ {
           root /home/www/;
    }
    ```

2. 虚拟目录和真实目录不同名

    上面的例子中alias设置的目录名和location匹配访问的path目录名一致，这样可以直接改成root目录配置；那要是不一致呢？
    再看一例：

    ```
    location /web/ {
          alias /home/www/html/;
    }
    ```

    访问http://www.wangshibo.com/web的时候就会去/home/www/html/下寻找访问资源。
    这样的话，还不能直接改成root目录配置。
    如果非要改成root目录配置，就只能在/home/www下将html->web（做软连接，即快捷方式），如下：

    ```
    location /web/ {
         root /home/www/;
    }
    
    # ln -s /home/www/web /home/www/html       //即保持/home/www/web和/home/www/html内容一致
    ```

## 小结

所以，一般情况下，在nginx配置中的良好习惯是：

1. 在location /中配置root目录；
2. 在location /path中配置alias虚拟目录。