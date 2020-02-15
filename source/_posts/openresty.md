---
title: OpenResty使用
date: 2016-06-06 19:58:27
categories: [OpenResty,Nginx]
tags: [OpenResty,Nginx]
visitors:
---
### 1.安装编译
{% link OpenResty官网下载安装包openresty-1.x.x.x.tar.gz http://openresty.org/cn/download.html openResty %}
{% codeblock lang:bash %}
#解压
tar -xzvf openresty-1.x.x.x.tar.gz
#安装
./configure
make
make instal
#启动
sudo service openresty -c /usr/local/openresty/nginx/conf/nginx.conf
{% endcodeblock %}
<!-- more -->
### 2.配置文件
{% codeblock lang:bash %}
#配置文件地址：/usr/local/openresty/nginx/conf/nginx.conf
#修改
server {
        listen 80;
        server_name zhoufangy.github.io, www.zhoufangy.github.io;

        location / {
            proxy_pass              http://0.0.0.0:4000/;
            proxy_redirect          off;
            proxy_set_header        X-Real-IP       $remote_addr;
            proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        }
}
{% endcodeblock %}
### 3.加载重新配置的nginx文件并启动
{% codeblock lang:bash %}
sudo /usr/local/openresty/nginx/sbin/nginx  -c /usr/local/openresty/nginx/conf/nginx.conf
{% endcodeblock %}
### 3.管理openresty
{% link google code https://fzrxefe.googlecode.com/files/openresty.init.d.script [external] [OpenResty] %}下载init.d 重命名openresty，放到服务器/etc/init.d目录下
{% codeblock lang:bash %}
sudo chmod +x /etc/init.d/openresty
sudo update-rc.d openresty defaults
#启动Nginx:
sudo service openresty start
#停止Nginx:
sudo service openresty stop
#重启Nginx:
sudo service openresty restart
#测试配置文件:
sudo service openresty test
{% endcodeblock %}
