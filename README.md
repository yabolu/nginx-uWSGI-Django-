# nginx-uWSGI-Django-

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

##### 3. 启动停止

```
//启动
/usr/local/nginx/sbin/nginx -c "nginx配置文件名"
//我这里打算用nginx监听8000端口, 所以我以/usr/local/nginx/conf/nginx.conf为模板创建配置文件nginx_8000.conf
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx_8000.conf

//停止
/usr/local/nginx/sbin/nginx -s stop
/usr/local/nginx/sbin/nginx -s quit

//重新加载配置文件
/usr/local/nginx/sbin/nginx -s reload
```

##### 4. 使用uWSGI启动Django 项目
我的项目结构如图所示:

![项目结构图片](https://github.com/yabolu/nginx-uWSGI-Django-/blob/master/images/project.png)



```
1. 将settings.py中的DEBUG 改为False, DEBUG = False

2. 配置uwsgi.ini
//配置方式一:
[uwsgi]
chdir = .
wsgi-file = db_full_stack_platform/wsgi.py
processes = 4
threads = 4
master = true   #主进程方式
socket=%(chdir)/uwsgi/uwsgi.sock
daemonize=%(chdir)/uwsgi/uwsgi.log
stats=%(chdir)/uwsgi/uwsgi.status
pidfile=%(chdir)/uwsgi/uwsgi.pid
vacuum=true #退出uWSGI后, 清除pid, status, sock文件

//配置方式二:
[uwsgi]
chdir = .
wsgi-file = db_full_stack_platform/wsgi.py
processes = 4
threads = 4
master = true   #主进程方式
socket=:8888
daemonize=%(chdir)/uwsgi/uwsgi.log
stats=%(chdir)/uwsgi/uwsgi.status
pidfile=%(chdir)/uwsgi/uwsgi.pid
vacuum=true #退出uWSGI后, 清除pid, status
```

为了更加方便的启动, 停止项目, 我写了shell脚本

```
//start.sh
~/.pyenv/versions/venv370/bin/uwsgi --ini ./uwsgi.ini

//stop.sh
~/.pyenv/versions/venv370/bin/uwsgi --stop ./uwsgi/uwsgi.pid

//reload.sh
~/.pyenv/versions/venv370/bin/uwsgi --reload ./uwsgi/uwsgi.pid

//-----------------------这里是分割线-----------------------
//启动项目
sh start.sh

//关闭项目
sh stop.sh

//重启项目
sh reload.sh
```

##### 5. nginx 关联uWSGI

修改配置文件`nginx_8000.conf`

```
//配置方式一:
server {
    listen 8000;
    server_name 127.0.0.1;
    location / {
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

//配置方式二:
server {
    listen 8000;
    server_name 127.0.0.1;
    location / {
        include uwsgi_params;
        uwsgi_pass 127.0.0.1:8888;
    }
    location /media {
        alias /dbtool/python_software/dbyw-python3/db_full_stack_platform/media;
    }
    location /static {
        alias /dbtool/python_software/dbyw-python3/db_full_stack_platform/static;
    }
}
```

##### 6. 启动项目并验证

启动nginx

```
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx_8000.conf
```

验证:

![验证](https://github.com/yabolu/nginx-uWSGI-Django-/blob/master/images/itworked.png)



