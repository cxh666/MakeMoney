[TOC]

# flaks+gunicorn+supervisor+nginx

- flask：python的服务器框架
- gunicorn：webservice，WSGI的容器
- supervisor：进程管理工具
- nginx：高性能web服务器

## flask

flask的教程详见flask.md



## gunicorn

flask自带的服务器虽然可以启动web服务，但是性能较差，一般用于调试。所以使用gunicorn作为wsgi的容器，用来不熟python。



## supervisor

管理系统进程

1. 安装：`pip install supervisor`

2. 生成配置文件：`echo_supervisord_conf > supervisor.conf`

3. 修改conf配置文件（文件底部）：

   ```
   [program:myapp]
   command=/var/www/server/venv/bin/gunicorn -w4 -b0.0.0.0:2170 myapp:app    ; supervisor启动命令
   directory=/var/www/server                                                ; 项目的文件夹路径
   startsecs=0                                                                             ; 启动时间
   stopwaitsecs=0                                                                          ; 终止等待时间
   autostart=false                                                                         ; 是否自动启动
   autorestart=false                                                                       ; 是否自动重启
   stdout_logfile=/var/www/server/log/gunicorn.log                           ; log 日志
   stderr_logfile=var/www/server/log/gunicorn.err  
   ```

4. `mkdir log`

5. 基本命令：

   ```
   supervisord -c supervisor.conf                             通过配置文件启动supervisor
   supervisorctl -c supervisor.conf status                    察看supervisor的状态
   supervisorctl -c supervisor.conf reload                    重新载入 配置文件
   supervisorctl -c supervisor.conf start [all]|[appname]     启动指定/所有 supervisor管理的程序进程
   supervisorctl -c supervisor.conf stop [all]|[appname]      关闭指定/所有 supervisor管理的程序进程
   ```

   

## Nginx

高性能HTTP和反向代理服务器，在高并发方面表现不错

1. 安装：`sudo apt-get install nginx`

2. 开启关闭命令：

   ```
   sudo /etc/init.d/nginx restart // 重启
   sudo /etc/init.d/nginx start //开启
   sudo /etc/init.d/nginx stop //关闭
   ```

3. 配置nginx

   ```
   cd /etc/nginx/sites-available/default
   cd /etc/nginx/sites-enabled/default
   ```

4. 修改的default文件：

   ```
   server {
     #侦听80端口
       listen 80;
   #定义使用www.xx.com访问
       server_name www.app.com; // 或则是地址(http://118.89.235.150/)
       client_max_body_size 10M;
    
      #设定本虚拟主机的访问日志
       access_log logs/app.log main;
    
     #默认请求
       location / {
           #请求转向本机ip:8888
           proxy_pass http://0.0.0.0:8000;
           proxy_redirect off;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       }
       #配置静态文件转发
       location ~.*(js|css|png|gif|jpg|mp3|ogg)$ {
           root /home/zhoujianghai/temp/data/app/medias/;
       }
       #配置静态页面转发
       location ~.*(html)$ {
           root /home/zhoujianghai/temp/data/app/app_static_pages/;
       }
   }
   
   ```

   