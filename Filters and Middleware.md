# Filter and Middleware

有时候，每个handler都要写一些重复的代码，写多了就觉得烦。在Django中，有middleware来解决这种问题，而在Bamboo中，有强大的Filter/Middleware来解决。

## 规则

### 写filter函数

一个filter函数，就是一个普通的lua函数，但参数表是固定的，为`(web, req, extra)`，两个都是lua table。并且要求返回值必须是`boolean[, table]`，第一个返回值，布尔量，必须；第二个table, 可选。比如：

	local function filter_A( web, req, extra )
		.....
		return true, extra_table
	end
	
	local function filter_B( passedin, extra )
		.....
		return true, extra_table
	end

### 配置URL映射

比如，原来没有添加Filter的URL映射如下：

	['/path/a/'] = handler_A

现在，要使用刚注册的那个filter，写成如下形式：

	['/path/a/'] = { filter_A, filter_B, handler_A }

则在调用`handler_A`来处理请求之前，会先依次执行`filter_A`和`filter_B`两个函数。并且

- `filter_A`返回的第二个值`extra_table` table，将作为`filter_B`的第三个参数`extra`传到`filter_B`中去；
- `filter_B`返回的第二个值`extra_table` table，将作为`handler_A`的第三个参数`extra_params`传到`handler_A`中去；
- `filter_A`与`filter_B`中只要有一个返回`false`，都将终止整个过程；
- 可以看到，bamboo只提供了前置filter函数。因为后置filter用得不多。对于这种情况，可以使用finish函数。

## 中间件

### 项目总入口与总出口
在`handler_entry.lua`中，`init`和`finish`是两个特殊的函数名，在定义时，它们前面不加`local`，即如下所示：
	
	function init(web, req, extra)
	
    return true, extra_table
	end

	function finish(web, req, extra)
	
    return true, extra_table
	end

那么，这个`init`函数就将是此bamboo工程所有handler函数的总入口。`finish`函数就是工程所有handler函数的总出口。

### 模块总入口与总出口
与全局的总入口与总出口类似，在模块中，`init`和`finish`也是两个特殊的函数名，在定义时，它们前面不加`local`，即如下所示：

	module(..., package.seeall)
	
	function init(web, req, extra)
    
    return true, extra_table
	end

	function finish(web, req, extra)
	
    return true, extra_table
	end

那么，这个`init`函数就是此模块下所有handler函数的总入口。`finish`函数就是此模块下所有handler函数的总出口。

所有写在这些总入口和出口中的代码，都可被视作中间件。

### 中间件
bamboo中的中间件（middleware），其实就是普通的lua代码或函数。它能够对传入的web，req, extra这些参数进行处理。为后面的handler提供预处理的一些东西。

因此，前面提到的filter，也是中间件的一种形式。而更常用的形式就是写在全局和各模块中的init及finish函数中的代码。

综上所述，我们得到了Bamboo的一个完整自洽的Filters体系。


## Bamboo的Filter体系完整流程图

![Filter体系流程图](/daogangtang/bamboo_doc_cn/blob/master/images/filter_params_passing.png?raw=true)