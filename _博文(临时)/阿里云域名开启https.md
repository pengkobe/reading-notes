# 阿里云域名开启HTTPS

## Why
* 保证数据安全
* 大公司推进 
  - 2017年1月1日起，苹果 App Store 中的所有 App 都必须启用 HTTPS 连接网络，否则将无法在应用商店上架
  - 2017年1月1日起，谷歌Chrome 56 版本浏览器将对未进行HTTPS加密的网页提出警报  

## 申请步骤
登录阿里云，进入控制台( 云盾 -> 证书服务 )，有专业证书( 好几千一年 )与免费证书( Symantec )可够选择,听说腾讯云有自己
做证书而不依赖第三方服务( 本人未测试 )。  
你要做的就是填好资料点击下一步直到签发( 域名解析时需要添加相关记录，你可以勾选系统自动添加 )。    
对于已签发的证书，下载后解压可以得到两个文件，一个 key 结尾，是私钥，一个 pem 结尾，是公钥。 

##  安装证书
下载证书页面有好几种证书可供选择，针对不同的服务器选择不同的证书。本认阿里云服务器有使用到 Nginx，于是选择下载 Nginx 版证书。
按照官方教程进行配置即可：

1. 证书文件xx.pem，包含两段内容，请不要删除任何一段内容。
2. 如果是证书系统创建的CSR，还包含：证书私钥文件xx.key。
 - 在Nginx的安装目录下创建cert目录，并且将下载的全部文件拷贝到cert目录中。如果申请证书时是自己创建的CSR文件，请将对应的私钥文件放到cert目录下并且命名为xx.key；
 - 打开 Nginx 安装目录下 conf 目录中的 nginx.conf 文件，找到：
   ```bash
        # HTTPS server
        # #server {
        # listen 443;
        # server_name localhost;
        # ssl on;
        # ssl_certificate cert.pem;
        # ssl_certificate_key cert.key;
        # ssl_session_timeout 5m;
        # ssl_protocols SSLv2 SSLv3 TLSv1;
        # ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
        # ssl_prefer_server_ciphers on;
        # location / {
        #
        #
        #}
        #}
    ```
3. 将其修改为 (以下属性中ssl开头的属性与证书配置有直接关系，其它属性请结合自己的实际情况复制或调整)：  
   ```bash
    server {
        listen 443;
        server_name localhost;
        ssl on;
        root html;
        index index.html index.htm;
        ssl_certificate   cert/xx.pem;
        ssl_certificate_key  cert/xx.key;
        ssl_session_timeout 5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;
        location / {
            root html;
            index index.html index.htm;
        }
    }
    ```
4. 重启 Nginx。
5. 通过 https 方式访问您的站点，测试站点证书的安装配置。如遇到证书不信任问题，请查看[相关文档](https://help.aliyun.com/knowledge_detail/48023.html?spm=5176.2020520163.cas.41.0hNCZC)。


## Node 网站开启 HTTPS
在 your-ngx.conf 中添加以下配置即可
```bash
listen 443 ssl;
ssl_certificate      /path/to/your/xx.pem;
ssl_certificate_key  /path/to/your/xx.key;
```
添加完后，可以检查下配置是否正确
```
./usr/nginx/sbin/nginx -t
```


## 问题集锦

### 按照阿里云文档配置完成后结果显示缺少 ngx_http_ssl_module 
按照教程*非覆盖*安装( nginx 开启关闭命令可以在 nginx.conf 中进行配置 )
```bash
# 关闭 nginx, 也可以进入 sbin 目录执行 ./nginx -s stop 
# service nginx -s stop
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
# make install 会覆盖安装
make
# 检查是否编译成功
./usr/local/nginx/sbin/nginx -V
# 拷贝配置文件
cp objs/nginx /usr/local/nginx/sbin/
# 开启 nginx ( /etc/init.d/nginx start ), 也可以进入 sbin 目录执行 ./nginx
service nginx start 
# 查看是否正常监听80端口
netstat -lnp|grep nginx  
```

### 80 端口跳转至 443
可以在 nginx 配置中增加一个跳转配置:
```
server {  
    listen  192.168.1.111:80;  
    server_name test.com;  
    rewrite ^(.*) https://$server_name$1 permanent;
}  
```

### 为什么绿锁变灰锁
简单来说就是你的网页加载了跨站内容



## 参考
http://www.itnpc.com/news/web/146771636486934.html

