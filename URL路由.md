# URL路由

在[前一章](第一个工程.md)中，你可能已经在`handler_entry.lua`中看到了一个叫`URLS`的lua表，

	URLS = {
		['/'] = index,
		['index/'] = index,
	}

这个表就是bamboo的主URL路由配置表。它的作用是定义URL与处理函数之间的对应关系。

比如，上面的配置就定义了：'/'和'/index/'这两个URL路径都由`index`函数来处理。而`index`函数就是一个普通的lua函数。

在`handler_entry.lua`的URLS中，路径key的写法，需遵循以下规则：

- 是不是以 '/' 开头和结尾都没关系。因此，`index` 与 `/index` 与 `index/` 与 `/index/` 效果是一样的；
- 这个key可以是lua的正则表达式，支持完备的lua正则表达式语法；
- 支持:号前缀命名参数。

## 正则表达式参数
我们可以用正则表达式写url key，并支持捕获。比如：

    ['/path/%w+/'] = path_handler,
    ['/path/(%w+)/'] = path_handler2

上述两行，能捕获的url是相同的（`/path/xxx/`这种形式）。不过，对第二个url来说，在`path_handler2`中，能够使用如下形式的参数：

    req.REST[1] --> 'xxx'

而不需要自己去匹配了。

Bamboo中，最多支持 **6** 个这样的捕捉（`req.REST[1], req.REST[2], ... req.REST[6]`）。

## 冒号表达式命名参数
我们可以用冒号表达式写url key，实现命名参数。比如：

    ['/path/%w+/'] = path_handler,
    ['/path/:pid/'] = path_handler2

上述两行，能捕获的url是相同的（`/path/xxx/`这种形式）。不过，对第二个url来说，在`path_handler2`中，能够使用如下形式的参数：

    req.REST.pid --> 'xxx'

而不需要自己去匹配了。

Bamboo中，最多支持 **6** 个这样的命名参数。


## 注意
- 变量URLS前面一定不能加`local`关键字；
- 如果访问了一个URLS（及所有其它模块）中没有配置的URL，则bamboo会返回一个默认的 **404** 页面提示给客户端。

