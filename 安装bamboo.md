


以Ubuntu 10.04为例。从前到后依次执行。你也可以下载脚本包[bamboo_installer-v1.1.tar.gz](https://github.com/downloads/daogangtang/bamboo_doc_cn/bamboo_installer-v1.1.tar.gz)，解压后，运行可执行脚本便会自动安装（需确保系统已经安装了lua5.1，如果没有，请使用 `sudo apt-get install lua5.1`来安装）。

## APT安装部分
##### 安装gcc等基本编译环境 

	sudo apt-get install build-essential 

##### 安装lua解释器，头文件，基本库，luarocks 

	sudo apt-get install lua5.1 liblua5.1-0 liblua5.1-0-dev luarocks 

##### 安装git, uuid-dev等，被zeromq及后面的程序需要 

	sudo apt-get install uuid-dev git-core 

## 编译安装部分
##### 安装libzmq 
到 [zeromq官网](http://www.zeromq.org/) 下载最新稳定版的安装包，比如是zeromq-2.1.7.tar.gz。（如果以前安装过老版的libzmq库，请手动将其删除，一般在/usr/local/lib/和/usr/lib/下面） 

	tar xvf zeromq-2.1.7.tar.gz 
	cd zeromq-2.1.7 
	./configure       		# 这个过程如果发现什么依赖不完全的，请apt-get手动装上 
	make 
	sudo make install     	# 会将zmq库文件安装到/usr/local/lib/下面 
	sudo ldconfig           # 执行这个，把库路径加入到缓存中去 

##### 安装redis 
到[redis官网](http://redis.io)上下载最新稳定版的安装包，比如是：redis-2.2.2.tar.gz 

	tar xvf redis-2.2.2.tar.gz 
	cd redis-2.2.2 
	make 
	sudo make install 

编辑配置文件 /etc/redis.conf，如果没有此文件就把当前redis源代码包中的redis.conf文件拷贝到/etc/下面去。配置文件可根据实际需求做更改。我的配置文件在[这里](others/redis.conf.md)，供参考：

## rocks依赖包安装部分 

在安装rocks包前，打开/etc/luarocks/config.lua 

	sudo vi /etc/luarocks/config.lua 
	添加下面一行到文件末尾 
	variables['CFLAGS'] = "-O2 -fPIC" 

保存，退出。 

下面开始执行rocks依赖包的安装。 

	sudo luarocks install lpeg 
	sudo luarocks install lua_signal 
	sudo luarocks install luaposix 
	sudo luarocks install luasocket 
	sudo luarocks install md5 
	sudo luarocks install lunit 
	sudo luarocks install telescope 

## git源码库安装部分 

下面执行git源码库的安装（以下命令如有权限问题，请在前面加sudo）。

在用户主目录下建立一个目录 GIT，cd进去，执行：

	mkdir ~/GIT
	cd GIT

##### 安装monserver

	git clone git://github.com/daogangtang/monserver.git

##### 安装lua-zmq

	git clone git://github.com/iamaleksey/lua-zmq.git 

##### 安装lgstring

	git clone git://github.com/daogangtang/lgstring.git 
	
##### 安装monserver-lua 

	git clone git://github.com/daogangtang/monserver-lua.git 

##### 安装redis-lua 

	git clone git://github.com/nrk/redis-lua.git 

##### 安装tnetstrings.lua 

	git clone git://github.com/jsimmons/tnetstrings.lua.git 

##### 安装luajson 

	git clone git://github.com/daogangtang/luajson.git 
	
##### 安装lglib 

	git clone git://github.com/daogangtang/lglib.git 

##### 安装bamboo 

	git clone git://github.com/daogangtang/bamboo.git 
	cd bamboo && sudo make install && cd ..

## 测试 

参见[第一个工程](第一个工程.md) 。

如果从前到后都顺利执行完毕的话，整个系统到此便安装成功了。 
