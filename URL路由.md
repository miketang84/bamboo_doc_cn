# URL路由

在[前一章](第一个工程.md)中，你可能已经在`handler_entry.lua`中看到了一个叫`URLS`的lua表，

	URLS = {
		['/'] = index,
		['index/'] = index,
	}

这个表就是bamboo的主URL路由配置表。它的作用是定义URL与处理函数之前的对应关系。比如，上面的配置就定义了：'/'和'/index/'这两个URL路径都由index函数来处理。而index函数就是一个普通的lua函数。

在`handler_entry.lua`的URLS中，路径key的写法，需遵循以下规则：

- 是不是以'/'开头和结尾都没关系，因此，`index` 与 `/index` 与 `index/` 与 `/index/` 效果是一样的；
- key中，允许的字符范围为：`0-9 a-z A-Z - _ % /`；
- 这个key可以是lua的正则表达式，支持完备的lua正则表达式。


### 注意
- 变量URLS前面一定不能加`local`关键字；
- 如果访问了一个URLS（及所有其它模块）中没有配置的URL，则bamboo会返回一个 **404** 页面给客户端。
