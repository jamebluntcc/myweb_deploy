# Web Deploy

虽然flask自带web server但是却不适用于生产,对于web服务器我们使用uWSGI。nginx用于反向代理处理外网对web服务器的访问。

## 部署前的准备

由于我们使用flask作为后台web框架所以最好采用virtualEnv进行python环境的配置,关于virtualEnv的使用详情[访问这里]('https://pypi.python.org/pypi/virtualenv')

- 安装uwsgi:`sudo pip install uwsgi`
- 安装nginx:`sudo apt-get install nginx` 启动nginx`sudo /etc/init.d/nginx start`

  ## 进行配置

   和常规的配置不同,我们采用的是socket进行通信:

打开/etc/nginx/conf.d/default.conf进行如下配置:

```
fastcgi_connect_timeout 300;
fastcgi_send_timeout 300;
fastcgi_read_timeout 300;  # if the 4 parameters below is too small, will cause timeout 504
fastcgi_buffer_size 256k;
fastcgi_buffers 16 256k;
fastcgi_busy_buffers_size 512k;
fastcgi_temp_file_write_size 512k;

server {
    listen       80;
    server_name  192.168.0.113; #your server_name ip or web-site domain name
    charset utf-8;

    location / {
        include uwsgi_params;
        uwsgi_pass unix:///home/chencheng/my_test_flask_app/uwsgi.sock;
        uwsgi_read_timeout 1800;
        uwsgi_send_timeout 300;
        proxy_read_timeout 300;
    }
}
```

在配置好nginx后,进行uwsgi的配置。在自己的web application下新建uwsgi.ini的配置文件,配置信息如下:

```
[uwsgi]
socket = /home/chencheng/my_test_flask_app/uwsgi.sock #your socket file
chmod-socket = 666
master = true

processes = 8
workers = 2

harakiri = 60
harakiri-verbose = true
#limit-post = 2097152
post-buffering = 8192

buffer-size=65535


max-requests = 10000

reload-on-as = 128
reload-on-rss = 96
no-orphans = true

log-slow = true

chdir = /home/chencheng/my_test_flask_app #your application's path
module = my_test #main python file
callable = app #app name
#virtualenv = /home/public/python_packages/lib/python/
pythonpath = /usr/bin/python #python path
```

其中uwsgi的配置文件中最重要的是:

- sock file
- chdir
- module
- callable
- pythonpath

运行uwsgi:`sudo uwsgi --ini /path/to/your/uwsgi.ini file`最后将nginx服务器重新启动一次:`sudo /ect/init.d/nginx restart`在浏览器上访问服务器的ip出现`haha`表明整个部署成功。在整个配置中最重要的是保证自己的uwsgi.sock文件在nginx、uwsgi中相同。
