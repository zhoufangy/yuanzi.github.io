---
title: 配置https
date: 2016-06-15 11:05
categories: [HTTPS,Nginx]
tages: [HTTPS,Nginx]
visitors:
---
### 1.申请SSL证书
WoSign证书申请：阿里云/官网->注册->下载密钥
### 2.新增zhoufangyuan.conf
{% codeblock lang:bash %}
# 利用HSTS实现HTTP->HTTPS
map $scheme $hsts_header {
    https   max-age=31536000;includeSubdomains; preload;
}        
server {
        listen 80; 
        listen 443 ssl; 
        server_name zhoufangyuan.me;
        ssl on;

        #指定证书文件 
        ssl_certificate      /usr/local/openresty/nginx/sslkey/cert.pem;
        ssl_certificate_key  /usr/local/openresty/nginx/sslkey/cert.key;
    
        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;
    
        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;

        location / {
                root /home/www/zhoufangyuan.me/;
                index  index.html index.htm;
                }
        #增加响应头，浏览器自动以HTTPS访问
        add_header Strict-Transport-Security $hsts_header;

        }
{% endcodeblock %}
<!--more -->
### 3.将zhoufangyuan.conf引入ngnix.conf
{% codeblock lang:bash %}
include vhost/*.conf;
{% endcodeblock %}
### 4.重新加载ngnix
{% codeblock lang:bash %}
sudo service openresty reload
{% endcodeblock %}
### 5.HTTP->HTTPS几种方法
#return 301
{% codeblock lang:bash %}
server {
        listen 80; 
        server_name  zhoufangyuan.me;
        return 301 https://$server_name$request_uri;
       }
{% endcodeblock %}
#rewrite重写
{% codeblock lang:bash %}
server {
        listen 80; 
        server_name  zhoufangyuan.me;
        rewrite ^(.*)$ https://$server_name$1 permanent;
       }
{% endcodeblock %}
#error_page 497
{% codeblock lang:bash %}
   server {  
    listen 443;  #ssl端口  
    listen 80;   #用户习惯用http访问，加上80，后面通过497状态码让它自动跳到443端口  
    server_name  zhoufangyuan.me;  
    #为一个server{......}开启ssl支持  
    ssl    on;  
    #指定证书文件   
    ssl_certificate      /usr/local/openresty/nginx/sslkey/cert.pem;
    ssl_certificate_key  /usr/local/openresty/nginx/sslkey/cert.key;
    #让http请求重定向到https请求   
    error_page 497  https://$host$uri?$args;  
} 
{% endcodeblock %}
#index.html 刷新网页
{% codeblock lang:html %}
# index.html
<html>
<meta http-equiv="refresh" content="0;url=https://zhoufangyuan.me/">
</html>

# zhoufangyuan.conf

server {  
    listen 80;  
    server_name zhoufangyuan.me;  
      
    location / {  
        #index.html放在虚拟主机监听的根目录下  
        root /home/www/zhoufangyuan.me/;  
    }
    #将404的页面重定向到https的首页
    error_page  404 https://zhoufangyuan.me/;
}
{% endcodeblock %}
以上四种方法
https访问网站正常,http跳转https失败,curl访问正常,浏览器访问失败
# HSTS强制HTTPS访问
{% link 参考资料  https://trac.nginx.org/nginx/ticket/289#comment:3 参
考资料 %}

