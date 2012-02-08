## web和req对象

bamboo工程中的每个handler，必须接收两个固定的参数`web`和`req`，`web`是连接对象，里面封装有一些处理请求的方法，`req`是请求对象，内含请求的所有属性。每进来一个请求，都会生成一个新的`web`对象和`req`对象，供handler调用。

通常，使用`web:page(html_str)`来返回html内容给客户端，使用`web:json(lua_table)`来返回json表给客户端。

下面列出web object和req object的详细内容。

### Web Object

##### - `web:page(data, code, status, headers)`

	将字符串数据data返回给客户端。

##### - `web:html( html_tmpl, tbl)`

	渲染模板html_tmpl，返回给客户端。web:page(View('xxxx'){})的快捷形式。
	- html_tmpl  字符串  模板文件的名字
	- tbl        表      待渲染的参数 

##### - `web:json(data, ctype)`

	Encode `data` (type is `table`) as json string and response it to client. The `ctype` parameter is optional, you can use it to redefine content type of this response.

##### - `web:jsonError(err_code, err_desc)`

	One shortcut to `web:json`, used for ajax error response.

##### - `web:jsonSuccess(tbl)`

	One shortcut to `web:json`, used for ajax success response.

##### - `web:redirect(url)`

	Redirect to `url`.

##### - `web:method()`

	Return the method of this http request, usually is GET or POST

##### - `web:isAjax()`

	Return whether this http request is ajax request.

##### - `web:path()`

	Return the URI path of this http request.

##### - `web:ok(msg)`

	Used to response OK page quickly.

##### - `web:notFound()`

	Used to response 404 page quickly.

##### - `web:unauthorized()`

	Used to response 401 page quickly.

##### - `web:forbidden()`

	Used to response 403 page quickly.

##### - `web:badRequest()`

	Used to response 400 page quickly.

##### - `web:close()`

	Close this web connection.

##### - `web:error(data, code, status, headers)`

	Used to response an page quickly, and close this connection.

##### - `web:session()`

	Return session id of this connection.

##### - `web:getCookie()`

	Return cookie value of this request.

##### - `web:setCookie(cookie)`

	Set `set-cookie` flag in headers.

##### - `web:zapSession()`

	Make a new cookie code and set it to request headers.

##### - `web:loginRequired(reurl)`

	Used for website page need login to view, if not login, this method will redirect to url `reurl`, if `reurl` is missing, '/index/' is used.

--------------------------

### Request Object

Request object is a lua table in Bamboo, usually, when passed into handler, use `req` to refer this table. `req` usually contains the following attributes:

##### - `req.GET`

	GET请求传上来的参数放在这里面。

##### - `req.POST`

	POST请求传上来的参数放在这里面。
	
##### - `req.PARAMS`

	GET或POST请求传上来的参数放在这里面。对于req.GET和req.POST中有相同的key的情况，以req.POST中的值为准；

##### - `req.conn_id`

	The connection id passed into bamboo by mongrel2.

##### - `req.sender`

	The sender uuid.

##### - `req.path`

	URI be visited in this request.

##### - `req.session_id`

	The session id of this request.

##### - `req.body`

	The body string part of this request.

##### - `req.data`

	A table, into which we can put some key-value.

##### - `req.headers`

	The request headers, another table.

##### - `req.headers.connection`

	Usually is 'keep-alive'.

##### - `req.headers.x-forwarded-for`

	Usually is '127.0.0.1'.

##### - `req.headers.host`

	The host of this website.

##### - `req.headers.accept-charset`

	Something like 'GB2312,utf-8;q=0.7,*;q=0.7'.

##### - `req.headers.VERSION`

	Usually is 'HTTP/1.1'.

##### - `req.headers.METHOD`

	GET, POST, PUT or DELETE.

##### - `req.headers.referer`

	The previous page on which you click an anchor to make this request.

##### - `req.headers.accept`

	Something like 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8'.

##### - `req.headers.PATH`

	URI of this request.

##### - `req.headers.cookie`

	The cookie string of this request.

##### - `req.headers.cache-control`

	Something like 'max-age=0'.

##### - `req.headers.accept-language`

	Something like 'zh-cn,zh;q=0.5'.

##### - `req.headers.user-agent`

	Something like 'Mozilla/5.0 (X11; U; Linux x86_64; zh-CN; rv:1.9.2.8) Gecko/20100723 Ubuntu/10.04 (lucid) Firefox/3.6.8'

##### - `req.headers.URI`

	URI of this request.

##### - `req.headers.keep-alive`

	Something like '115'.

##### - `req.headers.accept-encoding`

	Something like 'gzip,deflate'.

-----------------------------------
PS：由于太常用了，我们把web和req导出成了**全局变量**，在bamboo工程中的每一个地方都可以访问到。

