
整体需求：

实现v2ray+ws+tcl的方式科学上网，同时满足wordpress个人站点的正常运行

实现方式，通过v2ray单独一个域名+个人站点的单独一个域名，再调整nginx的SNI等方式运行起来特别复杂，经过几次尝试发现很难实现。
遂改变实现方式，使用v2ray一键自动化脚本下的元素周期表的index.html的目录，将wordpress的所有文件放在这个目录下并作index.html为index.php的替换。


实现方式：
1.VULTR的VPS， 在UBUNTU 18.04环境下，运行 一键自动话v2ray+ws+tls的插件。
>提前需要注册域名，并做dns的解析

bash <(curl -sL https://raw.githubusercontent.com/hijkpw/scripts/master/centos_install_v2ray2.sh)

2.
2.下载最新wordpress解压后复制*.*安装到对应的目录home\wwwroot\3DCEList
wget https://wordpress.org/latest.tar.gz
tar zxf latest.tar.gz  \\官网下载最新wordpress
cd wordpress   \\进入wordpress解压后目录
cp -R * home\wwwroot\3DCEList  \\复制wordpress目录及文件到网站缺省目录
chown -R www-data:www-data *  \\目录下权限设置

3.安装mysql
apt install mysql-server

sudo mysql -u root -p
mysql>    CREATE DATABASE wordpress; \\创建数据库wordpress
mysql>    CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'password'; \\创建wordpressuser用户和密码password,可自行修改
mysql>    GRANT ALL ON wordpress.* TO 'wordpressuser'@'localhost' IDENTIFIED BY 'password' WITH GRANT OPTION; \\授予权限
    FLUSH PRIVILEGES; \\配置生效
    EXIT;

4.因v2ray一键包已经安装nginx
调整关闭apache2
systemctl stop apache2

5.安装并配置php
访问网址ip地址，如果出现welcome to nginx表明nginx正常使用。

sudo apt-get install php7.2-fpm　　　　//安装php-fpm中间连接软件，本次安装为php7.2

5.1 修改\etc\php\7.2\fpm\php.ini的文件
nano \etc\php\7.2\fpm\php.ini

找到并取消注释#, 设置为cgi.fix_pathinfo=0 //，将值“1”改为值“0”

5.2 对pool.d目录下面的www.conf这个文件进行编辑

        listen =127.0.0.1：9000

　　　　listen.allowed_clients　 = 127.0.0.1

　　　　pm.max.children = 50

　　　　pm.max_requests = 500

　　　　request_terminate_timout = 0

　　　　rlimit_files = 1024

修改完成以上参数后，启动php-fpm

　　 systemctl start php7.2-fpm　　　　//启动php-fpm

6. 配置nginx支持php

ps -ef | grep nginx
找到目前运行nginx的主配置目录，如果是缺省一键安装版本，应该是在\etc\nginx\conf下面的nginx.conf，
***但需要注意的是，因为里面有include \conf.d\*.conf 这句，所以对应的文件都会对配置有影响。

如下这段文字开始放在nginx.conf并不起作用，后来放在了\conf.d\v2ray.conf下面才生效

location ~ \.php$ {
　　　　　　root html;
　　　　　　fastcgi_pass 127.0.0.1:9000;
　　　　　　fastcgi_index index.php;
　　　　　　fastcgi_param SCRIPT_FILENAME home\wwwroot\3DCEList$fastcgi_script_name;\\这里改为网站的目录
　　　　　　include fastcgi_params;

　　　　　　} 

　　注意：添加的内容一定要在Server这个大的容器内
7. 配置wordpress的数据库
cp wp-config-sample.php wp-config.php \\将Sample复制一份为正式的wp-config.php
编辑wp-config.php将上面的mysql设置的wordpress数据库名字，用户，密码填入



8.systemctl reload nginx
  service nginx restart
  service mysql restart
  
9. https://www.example.com访问看是否可正常进入install.php

10. 最终的配置：

【\etc\nginx\conf\nginx.conf】
user  root;
worker_processes  3;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  4096;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.php index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #root html;
        # proxy_pass   http://127.0.0.1:9000;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        location ~ \.php$ {
            root           html;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  /home/wwwroot/3DCEList$fastcgi_script_name;
            include        fastcgi_params;
        }

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

include conf.d/*.conf;
}

[\etc\nginx\conf\conf.d\v2ray.conf]

    server {
        listen 443 ssl http2;
        ssl_certificate       /data/v2ray.crt;
        ssl_certificate_key   /data/v2ray.key;
        ssl_protocols         TLSv1.1 TLSv1.2 TLSv1.3;
        ssl_ciphers           TLS13-AES-256-GCM-SHA384:TLS13-CHACHA20-POLY1305-SHA256:TLS13-AES-128-GCM-SHA256:TLS13-AES-128-CCM-8-SHA256:TLS13-AES-128-CCM-SHA256:EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+ECDSA+AES128:EECDH+aRSA+AES128:RSA+AES128:EECDH+ECDSA+AES256:EECDH+aRSA+AES256:RSA+AES256:EECDH+ECDSA+3DES:EECDH+aRSA+3DES:RSA+3DES:!MD5;
        server_name www.example.com;
        index index.php index.html index.htm;
        root  /home/wwwroot/3DCEList;
        error_page 400 = /400.html;
        location /2eab28c4/
        {
        proxy_redirect off;
        proxy_pass http://127.0.0.1:32230;
        proxy_http_version 1.1;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;
        }
\\下面是新增加的，为了让index.php能够生效
   location ~ \.php$ {
            root           html;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  /home/wwwroot/3DCEList$fastcgi_script_name;
            include        fastcgi_params;
        }

}
    server {
        listen 80;
        server_name www.example.com;
        return 301 https://www.example.com$request_uri;
    }

【\home\wwwroot\3DCElist\wp-config.php】
....其他省略
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** MySQL database username */
define( 'DB_USER', 'wordpressuser' );

/** MySQL database password */
define( 'DB_PASSWORD', '你的password' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );
....其他不变

11.参考网站：

https://www.linuxidc.com/Linux/2019-10/160949.htm
https://www.hijk.pw/v2ray-one-click-script-with-mask/
https://www.ecsoe.com/archives/38.html
https://www.centos.bz/2018/07/%E4%B8%80%E6%AC%A1ubuntunginx%E6%90%AD%E5%BB%BAwordpress%E7%9A%84%E7%BB%8F%E5%8E%86/