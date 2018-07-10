---
title: Nginx建立多个server
date: 2018-07-08 16:53:39
tags:
categories:
- nginx
---
1. 在`/etc/nginx/conf.d/`中添加`conf`文件，如`myblog.conf`，最简内容如下：
```
server {
    listen       <port>;
    root  /var/www/myblog/;                 # root path
    index index.html index.htm index.php;   # default page
}
```
2. 检查`/etc/nginx/conf.d/download.conf`是否有`include /etc/nginx/conf.d/*.conf;`， 如果没有则添加。
3. 运行`nginx -t`检查配置文件是否正确。
3. 检查权限：
	1. 查看运行nginx的用户:
	```
    ps aux | grep "nginx: worker process" | awk '{print $1}'
    ```
    2. 对配置中的`root`目录，如上例中的`/var/www/myblog/`及其所有父目录和需要http访问的子目录，设置权限为755，对所有文件设为644。*（注意这里很多网上资料的解决方案是将nginx运行用户改为root用户，这样也能解决权限导致的403问题，但出于安全考虑不建议这样做。）*
4. 访问http://&lt;ip address&gt;:&lt;port&gt; 检查配置的正确性。







