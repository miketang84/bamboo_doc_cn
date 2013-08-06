## lgserver简介

[lgserver](https://github.com/daogangtang/lgserver)是一个简洁高效易定制的web服务器。

### 库依赖

它的库依赖有：
1. lua5.1;
1. libzmq  2.x;
1. lua-zmq;
1. luaposix;
1. lua-cmsgpack;
1. lua-http-parser;

### 与bamboo的通信接口

lgserver与bamboo通过两个zmq通道进行通信。流程如下：
1. lgserver通过通道A（采用PUSH绑定）将压缩后的req数据传给bamboo；
2. bamboo端使用PULL连接在通道A上，接收来自lgserver的消息。多个bamboo进程间采用负载均衡的方式与一个lgserver进行通信；
3. bamboo端业务逻辑处理完成后，使用通道B采用PUSH方式把消息推回给lgserver;
4. lgserver使用PULL从通道B上接收到数据后，将消息转发给外部（浏览器）连接。

lgserver传给bamboo的数据包的格式：

	{
		url=,			-- 请求的URL，含路径与query
		path=,			-- 请求的路径
		query_string=,	-- URL中query部分
		fragment=,		-- URL中#部分（按道理是不应该传上来的）
		method=,			-- http请求方法
		version=,		-- http版本
		body=,			-- http body
		headers={
			...			-- http头，lua table形式
		}, 
		data={
						-- 一些特殊数据，目前留空
		}, 
		meta={			-- lgserver中产生的meta数据，用于与bamboo端的通信
				conn_id=, 
				completed=true
		}
	}

bamboo端返回给lgserver的数据格式：

	{
		data =,  			-- string: body string to reply
		code =,			    -- number: http code to reply
		status =,		    -- string: http status to reply
		headers =,		    -- table hash: http response headers to reply
		conns =,			-- table list: http connections to receive this reply
		meta =				-- table hash: some other info to lgserver

	}


### lgserver的配置文件

	static_mm = { type="dir" }
	static_default = { type="dir" }

	handler_mm = { 
		type="handler", 
		send_spec='tcp://127.0.0.1:1234',
		recv_spec='tcp://127.0.0.1:1235'
	}

	server = {
		name="server1",
		bind_addr = "0.0.0.0",
		port=80,
		access_log="logs/access.log",
		error_log="logs/error.log",
		default_host="mm.com",
		hosts = { 
			{       
				name="mm.com",
				matching="mm.com",
				root_dir = "/home/xen/workspace/lgserver/tmp/",
				['max-age'] = 60,
				routes={
					--	['/'] = static_mm,
					['/'] = handler_mm,
					['/media/'] = static_mm,
					['/static/'] = static_mm,
					
				}
			},
		}
	}


### lgserver的静态文件服务说明

在上述配置示例中，host 'mm.com' 的routes下面，'/media/'绑定到`static_mm`中。而`static_mm`的定义是：
	
	static_mm = { type="dir" }
	
它表示，凡是以'/media/'开头的URL，都以静态文件形式进行服务。这个时候，lgserver会以host.root_dir为基目录，上例中是'/home/xen/workspace/lgserver/tmp/'，加上'/media/'，即在'/home/xen/workspace/lgserver/tmp/media/'下面去找对应的文件。如果有几级目录('/xx/xx/xx/xxx.html')，也会相应地找几级目录下的文件。

同理，凡是以'/static/'开关的URL，都在'/home/xen/workspace/lgserver/tmp/static/'下面去找对应的文件。

lgserver针对URL目录越界攻击已经做了算法防范，请放心使用。
