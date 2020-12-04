# nginx-安装配置

## 1.下载pcre
1、主页地址：http://www.pcre.org/     下载pcre-7.8.tar.bz22、解压缩：    把文件放到centos8中的/usr/src目录下， tar -zvxf pcre-7.8.tar.bz2
	进入解压目录，先执行./configure，再执行make && make install 检查并检查安装
	需要用到解压 命令tar -zvxf
	输入执行解压该解压包命令
	tar -zvxf pcre-8.37.tar.gz
	解压后，进入解压目录，
	先执行./configure
	
	其中编译命令./configure是用来检测你的安装平台的目标特征的。比如它会检测你是不是有CC或GCC,一般用来生成 Makefile，为下一步的编译做准备
	
	进入解压目录，输入make && make install,
	等待安装
	安装完成后，输入命令 rpm –qa pcre 
	可以查看到版本号，说明安装成功


### 2.安装openssl 和zlib
	输入命令
	yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel
	耐心等待安装即可
	时间可能有一点长，跟网速有关
	时间慢的可能十多分钟
	安装成功后，可以再次输入命令，可查看完成情况complete

## 3.Nginx安装
	到官网下载nginx安装包
	http://nginx.org/en/download.html
	与安装PCRE一样，
	解压NGINX压缩包
	输入执行解压该解压包命令
	tar -zvxf nginx-1.12.2.tar.gz 
	解压后，进入解压目录，
	先执行./configure
	然后安装make && make install
	不过在centos8中会出现错误，应该是版本问题，在centos7没有问题
	直接修改文件 vim src/os/unix/ngx_user.c，修改完记得保存
	然后使用vim /usr/src/nginx-1.12.2/objs/Makefile，同样记得保存
	删除-werror，保存退出
	再去执行安装命令，返回到解压目录下
	安装好以后运行nginx：在/usr/local/nginx/sbin 目录下执行 ./nginx
	直接访问该服务器查看运行是否成功


### 4.常用命令
	
	进入默认安装位置 /usr/local/nginx/sbin
	输入./nginx –h.
	-h help命令可以查看帮助信息
	查看服务是否启动成功
	ps –ef | grep nginx
	其中，快速停止命令
	/usr/local/nginx/sbin/nginx –s stop
	如果想关闭nginx，完整有序的停止nginx，保存相关信息
	可以使用命令Nginx –s quit


