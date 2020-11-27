# Nginx解决跨域问题

###### 什么是跨域问题？

浏览器从一个域名的网页去请求另一个域名的资源时，域名、端口、协议任一不同，都是跨域

![image-20201126151647800](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20201126151647800.png)



修改nginx的配置文件，如果你是默认安装，那么就在/usr/local/nginx/conf/nginx.conf这个文件中，在http模块当中加入以下代码

    server {
            listen       9001;
            server_name  localhost;
        location /panda {
            proxy_pass http://121.36.57.122:8081;
        }
        location /data {
            proxy_pass http://121.36.57.122:8080;
        }
        location /travel {
            proxy_pass http://121.36.57.122:8080;
        }
         location /upload {
            proxy_pass http://121.36.57.122:8080;
        }
    }
该代码意思，监听本地ip的9001端口，发现路径是/panda开头的就转发到

8081端口的静态资源，剩下的三个也是同样意思，都是监听转发到动态资源服务器，事实上这就已经解决了跨域问题，现在静态资源和动态资源暴露出来的效果就是在同一个端口，不存在跨域问题。然后修改页面访问数据接口的端口为9001，访问页面的端口也改为9001，这样就能成功访问了

![image-20201126151723779](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20201126151723779.png)