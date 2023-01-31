# [SSL证书格式PEM、CER、JKS、PKCS12](https://segmentfault.com/a/1190000042046457)

## 一、主流的SSL证书格式

根据不同的服务器以及服务器的版本，我们需要用到不同的证书格式，就市面上主流的服务器来说，大概有以下格式：

DER、CER，文件是二进制格式，只保存证书，不保存私钥。
PEM，一般是文本格式，可保存证书，可保存私钥。
CRT，可以是二进制格式，可以是文本格式，与 DER 格式相同，不保存私钥。
PFX P12，二进制格式，同时包含证书和私钥，一般有密码保护。
JKS，二进制格式，同时包含证书和私钥，一般有密码保护。

## 二、各服务器平台使用的证书格式

Nginx：pem
Apache：pem
IIS：pfx/pkcs12
Tomcat：jks

## 三、各证书格式详解

### 3.1、DER

该格式是二进制文件内容，Java 和 Windows 服务器偏向于使用这种编码格式。

OpenSSL 查看

openssl x509 -in certificate.der -inform der -text -noout
转换为 PEM

openssl x509 -in cert.crt -inform der -outform pem -out cert.pem

### 3.2、PEM

Privacy Enhanced Mail，一般为文本格式，以 -----BEGIN... 开头，以 -----END... 结尾。中间的内容是 BASE64 编码。这种格式可以保存证书和私钥，有时我们也把PEM 格式的私钥的后缀改为 .key 以区别证书与私钥。具体你可以看文件的内容。

这种格式常用于 Apache 和 Nginx 服务器。

OpenSSL 查看：

openssl x509 -in certificate.pem -text -noout
转换为 DER

openssl x509 -in cert.crt -outform der -out cert.der

### 3.3、CRT

Certificate 的简称，有可能是 PEM 编码格式，也有可能是 DER 编码格式。如何查看请参考前两种格式。

### 3.4、PFX

Predecessor of PKCS#12，这种格式是二进制格式，且证书和私钥存在一个 PFX 文件中。一般用于 Windows 上的 IIS 服务器。改格式的文件一般会有一个密码用于保证私钥的安全。

OpenSSL 查看：

openssl pkcs12 -in for-iis.pfx
转换为 PEM

openssl pkcs12 -in for-iis.pfx -out for-iis.pem -nodes

### 3.5、JKS

Java Key Storage，很容易知道这是 JAVA 的专属格式，利用 JAVA 的一个叫 keytool 的工具可以进行格式转换。一般用于 Tomcat 服务器。
参考链接：
[https://sslhow.com/openssl-pk...](https://link.segmentfault.com/?enc=tHl7geetZOzpp0jR2%2B90zg%3D%3D.R79Nu1O%2BSbKr7IOgoTCmvYEqc88apoxbsX1PbFMfaID179HQ4r94yZhnCcIkJHOX)