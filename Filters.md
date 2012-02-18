# Filters

有时候，每个handler都要写一些重复的代码，写多了就觉得烦。在Django中，有middleware来解决这种问题，而在Bamboo中，有强大的Filters来解决。

## 规则
Filters的规则概述如下：

1. 所有的handler现在默认依次接收3个参数`web, req, extra_params`。写成 `local function xxxxxx(web, req, extra_params)`的形式；
2. 如果在写URL规则的时候，直接是 `['/some/url/'] = some_handler_func` 的形式，则上述3个参数的前两个有真实的值，`extra_params`只是一张空表；
3. 如果在写URL规则的时候，是 `['/some/url/'] = { handler = some_handler_func, filters = { 'filter1', 'filter2'}` 的形式，则可以通过最后一个filter返回`true, extra_params`这两个参数来向handler中传递这个额外参数，注意，返回的参数`extra_params`应该为一个table；
4. 每一个filter接收两个参数，`passed_in_params, extra_params`，写成 `local function some_filter(passedin, extra)`的形式，`passedin`是在filter引用描述的时候定义的，是一个字符串的列表，`extra_params`最开始（传入第一个filter）为空表。返回两个参数，`boolean, extra`，其中，第一个为true或false，是必须的；第二个是可选的，如果有的话，应该是一个table；
5. fitler的上述输入输出，为filter chains的建立提供的完备的基础。在filter chains之间传递数据，以及在filter chains和handler之间传递数据，都是用这个额外表参数`extra_params`实现的。`extra_params`就好像是一艘船，每到一个地方就可以驮一些东西上来，经过重重关卡，最终随水流把货物送到handler中去；
6. handler在不需要`extra_params`参数时，可以不写参数，直接`local function xxxx()` 的形式。但建议把web, req都写上，当需要用到第三个参数的时候，必须把3个参数都写上；
7. 在filter链中任何一个环节（函数）返回`false`，整个链就将断掉。

如上规则，实际上定义了一套非常灵活的体系，使得filter这种东西，可以完成很多事情，并且有累积性。

## 如何使用filter

要使用filter，需要做3件事情：

1. 写filter函数；
2. 注册此filter；
3. 在URL映射表中写上对filter的引用和要传递的参数。

下面分别讲述。

### 写filter函数

一个filter函数，就是一个普通的lua函数，但参数表是固定的，为`passedin, extra`，两个都是lua table。并且要求返回值必须是`boolean[, table]`，第一个返回值，布尔量，是必须的；第二个table, 如果不需要，可以省略。比如：

	local function filter_A( passedin, extra )
		.......
		return true, extra
	end
	
	local function filter_B( passedin, extra )
		.......
		return true, extra
	end

### 注册filter函数

写好filter函数后，在使用前，需要将其注册，注册一般放在`handler_entry.lua`中。比如：

	bamboo.registerFilter('filter_A', filter_A)
	bamboo.registerFilter('filter_B', filter_B)
	

如果图省事，写filter函数和注册filter函数这两步可以合到一步来做。

### 配置URL映射

比如，原来没有添加filter的URL映射如下：

	['/user/info/'] = handler_userinfo

现在，要使用刚注册的那个filter，写成如下形式：

	['/user/info/'] = { handler = handler_userinfo, filters = { "filter_A: param1 param2 param3 param4", "filter_B: param5 param6 param7 param8" } }

则在调用`handler_userinfo`来处理请求之前，会先依次执行`filter_A`和`filter_B`两个函数。并且

- `param1 param2 param3 param4`将作为参数被分割存放到`passedin` table中并传给`filter_A`作为第一个参数。因此，`param1 param2 param3 param4`一般都为常量字符串；
- 同理`param5 param6 param7 param8`将作为参数被分割存放到`passedin` table中并传给`filter_B`作为第一个参数；
- `filter_A`返回的第二个值`extra` table，将作为`filter_B`的第二个参数`extra`传到`filter_B`中去；
- `filter_B`返回的第二个值`extra` table，将作为`handler_userinfo`的第三个参数`extra_params`传到`handler_userinfo`中去；
- `filter_A`与`filter_B`中只要有一个返回`false`，都将终止整个过程。

## Filter的种类和辅助函数

Filter分为 `prev filter`, `post filter`和 `隐性 filter`。prev filter与post filter本质上是一样的，注册方式也一样，只是在URL映射写法上有不同，比如上例，我们还有另外两个filter，名为`filter_C`和`filter_D`：

	['/user/info/'] = { 
		handler = handler_userinfo, 
		filters = { "filter_A: param1 param2 param3 param4", "filter_B: param5 param6 param7 param8" },		-- prev filters
		post_filters = { "filter_C: param9", "filter_D: param10" }, 										-- post filters
	}

Post filter在handler被执行后调用，也可以形成filter chain.

另外，Bamboo还提供了几个辅助filter函数。分别是：

- `bamboo.registerFilters(filter_table)`

	批量注册filters。所有的filters按列表形式写在filter_table里面。比如：
	bamboo.registerFilters {
		{'filter_A', function () ....  end},
		{'filter_B', function () ....  end},
		{'filter_C', function () ....  end},
		{'filter_D', function () ....  end},
	}

- `bamboo.getFilterByName( filter_name )`

	根据 filter_name 来找到 filter 的定义函数。 

- `bamboo.executeFilters( filters, params )`

	批量顺序执行指定的filters。
	- filters	一个filter名称（及常量参数）列表，基本元素为字符串。如： {'filter_A', 'filter_B', 'filter_C', 'filter_D'}
	- params	传递给第一个filter函数的额外参数，要求是一个table。可省略。

## 隐性Filters

Bamboo中，有如下几个隐性Filters。

### 项目总入口与总出口
在`handler_entry.lua`中，`init`和`finish`是两个特殊的函数名，在定义时，它们前面不加`local`，即如下所示：
	
	function init()
	
	end

	function finish()
	
	end

那么，这个`init`函数就将是此bamboo工程所有handler函数的总入口。`finish`函数就是工程所有handler函数的总出口。

### 模块总入口与总出口
与全局的总入口与总出口类似，在模块中，`init`和`finish`也是两个特殊的函数名，在定义时，它们前面不加`local`，即如下所示：

	module(..., package.seeall)
	
	function init()
	
	end

	function finish()
	
	end

那么，这个`init`函数就是此模块下所有handler函数的总入口。`finish`函数就是此模块下所有handler函数的总出口。

综上所述，我们可以得到Bamboo的一个完整自洽的Filters体系。

## Bamboo的Filter体系完整流程图

![Filter体系流程图](/daogangtang/bamboo_doc_cn/blob/master/images/filter_params_passing.png?raw=true)