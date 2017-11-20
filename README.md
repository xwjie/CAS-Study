# CAS-Study
学习CAS单点登录框架

# 生成 cas war 包

去 [https://github.com/apereo/cas-overlay-template](https://github.com/apereo/cas-overlay-template) 下载工程。

执行 `build package` , 在 `target` 生成了 `cas.war` 。200多m。copy到tomcat的webapp目录。


#  tomcat 配置

生成https证书

```
C:\Program Files\Java\jdk1.8.0_101\bin>keytool -genkey -alias cas -keyalg RSA -keystore D:/Servers/cas.keystore
```

修改 tomcat server.xml，打开https配置。

```XML
    <Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
               maxThreads="150" SSLEnabled="true" scheme="https" secure="true"
               clientAuth="false" sslProtocol="TLS" 
               keystoreFile="D:/Servers/cas.keystore"   
               keystorePass="12345678"/>
```

启动tomcat，访问： https://localhost:8443/cas

# nignx 配置

## 创建证书
openssl使用apache\bin目录下的即可。

```
set OPENSSL_CONF=D:\Servers\Apache24\conf\openssl.cnf
```

创建服务器私钥，命令会让你输入一个口令：

```
$ openssl genrsa -des3 -out server.key 1024
```

创建签名请求的证书（CSR）：

```
$ openssl req -new -key server.key -out server.csr
```

>这里是域名：
Common Name (e.g. server FQDN or YOUR name) []:me.xiaowenjie.cn

在加载SSL支持的Nginx并使用上述私钥时除去必须的口令：

```
$ copy server.key server.key.org
$ openssl rsa -in server.key.org -out server.key
```

最后标记证书使用上述私钥和CSR：

```
$ openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```

把文件copy到nginx目录

```
D:\Servers\Apache24\bin>copy server.* d:\Servers\nginx-1.11.11.1-Lion\conf
server.crt
server.csr
server.key
server.key.org
```

## 配置nginx


修改Nginx配置文件，让其包含新标记的证书和私钥：

```
server {
	listen 80;
	listen 443 ssl http2;
	server_name  me.xiaowenjie.cn;
	ssl_certificate server.crt;
	ssl_certificate_key server.key;

	location / {
		root D:/OutPut/SpringBootUeditor/nashome/demo;
	}
}
```

>http2 支持http2

