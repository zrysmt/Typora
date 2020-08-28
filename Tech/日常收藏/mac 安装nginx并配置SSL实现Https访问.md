# mac 安装nginx并配置SSL实现Https访问

[![img](https://upload.jianshu.io/users/upload_avatars/2790249/169a94f81ed7.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)](https://www.jianshu.com/u/b6624224c411)

[milletmi](https://www.jianshu.com/u/b6624224c411)关注

2018.03.21 20:10:02字数 342阅读 4,293

## 一、安装



```javascript
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew install nginx 安装nginx
nginx -v 显示版本号
```

找到Nginx文件夹



```javascript
cd /usr/local/etc/nginx
```

启动nginx



```javascript
sudo nginx 
or 
nginx
```

关闭nginx



```javascript
sudo nginx -s stop
```

重启nginx



```javascript
sudo nginx -s reload
```

检查nginx配置



```javascript
nginx -t nginx.conf
or
sudo nginx -t nginx.conf
```

查日志



```javascript
cd /usr/local/var/log/nginx
tail -f access.log
tail -f error.log
```

## 二、配置

1、找到Nginx文件夹



```javascript
cd /usr/local/etc/nginx
```

2、openssl生成自签名证书

创建服务器私钥，命令会让你输入一个口令



```javascript
openssl genrsa -out server.key 1024
```

根据私钥生成证书申请,创建签名请求的证书（CSR）



```javascript
openssl req -new -key server.key -out server.csr
```

下面的选项至少写一个，才可以生成证书成功



```javascript
Country Name (2 letter code) []:ch
State or Province Name (full name) []:
Locality Name (eg, city) []:
Organization Name (eg, company) []:
Organizational Unit Name (eg, section) []:
Common Name (eg, fully qualified host name) []:
Email Address []:
```

在加载SSL支持的Nginx并使用上述私钥时除去必须的口令：



```javascript
$ cp server.key server.key.org

$ openssl rsa -in server.key.org -out server.key
```

最后标记证书使用上述私钥和CSR



```javascript
openssl x509 -req -in server.csr -out server.crt -signkey server.key -days 3650
```

4、配置nginx：修改/usr/local/etc/nginx/nginx.conf 文件



```javascript
location / {

root  html（当前静态文件的路径）;

index  index.html index.htm;

}
```

> 不过最好不要把配置写到/usr/local/etc/nginx/nginx.conf里面,而是写在当前目录下面的servers的文件夹下，创建不同的config更加清晰：
> 例如：servers下建立一个millet.conf
> 在millet.conf里面配置



```javascript
server {
        listen       443 ssl;
        server_name  static.millet.com;
         #server.crt和server.key都在nginx下面
        ssl_certificate      server.crt;
        ssl_certificate_key  server.key;

        location / {
            root   （当前静态文件的路径）;
            index  index.html index.htm;
        }
    }
```

ihost



```javascript
127.0.0.1 static.millet.com
```

#### 启动nginx



```javascript
sudo nginx
或者
sudo brew services start nginx
```

#### 停止nginx



```javascript
sudo nginx -s stop
或者
sudo brew services stop nginx
```

#### 重新加载配置文件



```javascript
sudo nginx -s reload
或者
sudo brew services restart nginx
```

#### 安装常见问题：

安装Nginx输入命令brew install nginx报错

You have not agreed to the Xcode license. Please resolve this by running:

sudo xcodebuild -license

nginx设置本地跨域

在mac 终端运行命令的时候会被提示没有同意xcode 证书 ，这个时候需要在Terminal中同意license

此时 输入命令 sudo xcodebuild -license 空格到最后 输入agree 就可以了