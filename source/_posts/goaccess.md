---
title: GoAccess使用
date: 2016-06-28 23:58:27
categories: [GoAccess,日志分析,Nginx]
tags: [GoAccess,日志分析,Nginx]
visitors:
---
### 关于GoAccess
{% link GoAccess https://goaccess.io/ GoAccess %}是一个开源的实时Web日志分析工具，能将有价值的HTTP统计数据生成可视化动态报告。
### 安装
{% codeblock lang:bash %}
#下载
wget http://tar.goaccess.io/goaccess-1.0.1.tar.gz
tar -zxvf goaccess-1.0.1.tar.gz
cd goaccess-1.0.1/
#安装依赖包
apt-get install libncursesw5-dev
apt-get install libgeoip-dev
./configure --enable-geoip --enable-utf8
#C系安装
make && make install
{% endcodeblock %}
<!-- more -->
### 使用
{% codeblock lang:bash %}
#修改goaccess.conf中必选项
vim /usr/local/etc/goaccess.conf
######################################
# Time Format Options (required)
######################################
time-format %H:%M:%S
######################################
# Date Format Options (required)
######################################
date-format %d/%b/%Y
######################################
# Log Format Options (required)
######################################
log-format %h %^[%d:%t %^] "%r" %s %b "%R" "%u"
{% endcodeblock %}
{% codeblock lang:bash %}
#处理需要分析的日志文件
cd /usr/local/openresty/nginx/logs/
goaccess -f access.log -o report.html --real-time-html --ws-url=goaccess.io
#Python创建简单HTTP服务用于访问
python -m SimpleHTTPServer
{% endcodeblock %}