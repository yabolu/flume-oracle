### redhat nginx 安装和配置

##### 1. 安装依赖包

```
yum install pcre-devel
yum install zlib zlib-devel
yum install openssl openssl-devel
```

##### 2. 安装nginx 

* 下载最新的的[nginx](http://nginx.org/en/download.html), 我下载的是 `nginx-1.14.2.tar.gz`

```
tar zxvf nginx-1.14.2.tar.gz
cd nginx-1.14.2
./configure --prefix=/usr/local/nginx
make
make install
```

##### 3. nginx 关联uWSGI

修改配置文件`nginx_8000.conf`

```
server {
    listen 8000;
    server_name 127.0.0.1;
    location / {
        uwsgi_read_timeout 500;#这里默认是60
        include uwsgi_params;
        uwsgi_pass unix:///dbtool/python_software/dbyw-python3/uwsgi/uwsgi.sock;
    }
    location /media {
        alias /dbtool/python_software/dbyw-python3/db_full_stack_platform/media;
    }
    location /static {
        alias /dbtool/python_software/dbyw-python3/db_full_stack_platform/static;
    }
}

```

##### 4. 启动项目并验证

启动nginx

```
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx_8000.conf
```










