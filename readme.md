
��������

ʵ��v2ray+ws+tcl�ķ�ʽ��ѧ������ͬʱ����wordpress����վ�����������

ʵ�ַ�ʽ��ͨ��v2ray����һ������+����վ��ĵ���һ���������ٵ���nginx��SNI�ȷ�ʽ���������ر��ӣ��������γ��Է��ֺ���ʵ�֡�
��ı�ʵ�ַ�ʽ��ʹ��v2rayһ���Զ����ű��µ�Ԫ�����ڱ���index.html��Ŀ¼����wordpress�������ļ��������Ŀ¼�²���index.htmlΪindex.php���滻��


ʵ�ַ�ʽ��
1.VULTR��VPS�� ��UBUNTU 18.04�����£����� һ���Զ���v2ray+ws+tls�Ĳ����
>��ǰ��Ҫע������������dns�Ľ���

bash <(curl -sL https://raw.githubusercontent.com/hijkpw/scripts/master/centos_install_v2ray2.sh)

2.
2.��������wordpress��ѹ����*.*��װ����Ӧ��Ŀ¼home\wwwroot\3DCEList
wget https://wordpress.org/latest.tar.gz
tar zxf latest.tar.gz  \\������������wordpress
cd wordpress   \\����wordpress��ѹ��Ŀ¼
cp -R * home\wwwroot\3DCEList  \\����wordpressĿ¼���ļ�����վȱʡĿ¼
chown -R www-data:www-data *  \\Ŀ¼��Ȩ������

3.��װmysql
apt install mysql-server

sudo mysql -u root -p
mysql>    CREATE DATABASE wordpress; \\�������ݿ�wordpress
mysql>    CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'password'; \\����wordpressuser�û�������password,�������޸�
mysql>    GRANT ALL ON wordpress.* TO 'wordpressuser'@'localhost' IDENTIFIED BY 'password' WITH GRANT OPTION; \\����Ȩ��
    FLUSH PRIVILEGES; \\������Ч
    EXIT;

4.��v2rayһ�����Ѿ���װnginx
�����ر�apache2
systemctl stop apache2

5.��װ������php
������ַip��ַ���������welcome to nginx����nginx����ʹ�á�

sudo apt-get install php7.2-fpm��������//��װphp-fpm�м��������������ΰ�װΪphp7.2

5.1 �޸�\etc\php\7.2\fpm\php.ini���ļ�
nano \etc\php\7.2\fpm\php.ini

�ҵ���ȡ��ע��#, ����Ϊcgi.fix_pathinfo=0 //����ֵ��1����Ϊֵ��0��

5.2 ��pool.dĿ¼�����www.conf����ļ����б༭

        listen =127.0.0.1��9000

��������listen.allowed_clients�� = 127.0.0.1

��������pm.max.children = 50

��������pm.max_requests = 500

��������request_terminate_timout = 0

��������rlimit_files = 1024

�޸�������ϲ���������php-fpm

���� systemctl start php7.2-fpm��������//����php-fpm

6. ����nginx֧��php

ps -ef | grep nginx
�ҵ�Ŀǰ����nginx��������Ŀ¼�������ȱʡһ����װ�汾��Ӧ������\etc\nginx\conf�����nginx.conf��
***����Ҫע����ǣ���Ϊ������include \conf.d\*.conf ��䣬���Զ�Ӧ���ļ������������Ӱ�졣

����������ֿ�ʼ����nginx.conf���������ã�����������\conf.d\v2ray.conf�������Ч

location ~ \.php$ {
������������root html;
������������fastcgi_pass 127.0.0.1:9000;
������������fastcgi_index index.php;
������������fastcgi_param SCRIPT_FILENAME home\wwwroot\3DCEList$fastcgi_script_name;\\�����Ϊ��վ��Ŀ¼
������������include fastcgi_params;

������������} 

����ע�⣺���ӵ�����һ��Ҫ��Server������������
7. ����wordpress�����ݿ�
cp wp-config-sample.php wp-config.php \\��Sample����һ��Ϊ��ʽ��wp-config.php
�༭wp-config.php�������mysql���õ�wordpress���ݿ����֣��û�����������



8.systemctl reload nginx
  service nginx restart
  service mysql restart
  
9. https://www.example.com���ʿ��Ƿ����������install.php

10. ���յ����ã�

��\etc\nginx\conf\nginx.conf��
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
\\�����������ӵģ�Ϊ����index.php�ܹ���Ч
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

��\home\wwwroot\3DCElist\wp-config.php��
....����ʡ��
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** MySQL database username */
define( 'DB_USER', 'wordpressuser' );

/** MySQL database password */
define( 'DB_PASSWORD', '���password' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );
....��������

11.�ο���վ��

https://www.linuxidc.com/Linux/2019-10/160949.htm
https://www.hijk.pw/v2ray-one-click-script-with-mask/
https://www.ecsoe.com/archives/38.html
https://www.centos.bz/2018/07/%E4%B8%80%E6%AC%A1ubuntunginx%E6%90%AD%E5%BB%BAwordpress%E7%9A%84%E7%BB%8F%E5%8E%86/