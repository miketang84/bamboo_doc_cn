# Mixin

熟悉RoR（Ruby On Rails）的人都知道，Ruby中有mixin和include的概念。大概意思就是可以定义一些相对独立的中间体mixin，里面可以包含属性、方法。在必要的时候，可以通过include指令将mixin包含到其它的class中，mixin中的属性和方法会自动注入到这个class中，就好像这个class中原生定义这些属性和方法似的。

这个机制，有助于将一些相对独立的功能独立出来，而不需要纳入到严密的类的继承体系中去。有助于简化设计、减轻开发者的压力。

在Bamboo中，也加入了类似的功能。这个功能是在bamboo所依赖的lglib中实现的。


## Mixin长什么样？

在Bamboo中，Mixin就是下面的这个样子。新增一个文件，比如mixin_example.lua，在里面写上

	module(..., package.seeall)
	
	return function (...)
		
		......
		
		return {
			__fields = {
				['example_field1'] = {},
				['example_field2'] = {},
			};
			
			init = function (self, t)
				if not t then return self end
				
				example_field1 = t.example_field1
				example_field2 = t.example_field2
				
				return self
			end;
		
			example_method1 = function (param1, param2)
			
				....
			end;
		
			example_method2 = function (param1, param2)
			
				....
			end;
		}
	end
	
上面就是一个完整的mixin了。可以看到，一个mixin是一个独立的lua模块文件。mixin由下面几部分构成：

1. 文件头部写上 module(..., package.seeall)；
2. 紧接着，直接 return 一个匿名函数，这个函数可以接收一些参数，这些参数是由include调用传进来的；
3. 在这个函数中，可以做一些操作。但最后，必须返回一个table，这个table中，有__fields, init, 或其它方法，就像传统的模型定义方法一样。它们都是可选的，视情况需要而定。在同一个mixin中的方法，总是试图操作本mixin中定义的这些新字段，而不要操作其它类或mixin中的字段和属性；

好了，mixin定义出来了，下面看一看怎么使用这个mixin。

## include mixin

比如，我们有下面这个Project模型，定义在文件project.lua中。

	module(..., package.seeall)
	
	local Model = require 'bamboo.model'
	
	local Project = Model:extend {
		__tag = 'Bamboo.Model.Project';
		__name = 'Project';
		__desc = 'Generitic Project definition';
		__indexfd = 'name';
		__use_rule_index=true;
		__fields = {
			['name'] = { required=true },	
			['intro'] = { required=true },
			['bugs'] = {foreign="Bug", st="MANY"}
		};
	
		init = function (self, t)
			if isFalse(t) then return self end
			
			-- auto fill non-foreign fields with params t
			local fields = self.__fields
			for k, v in pairs(t) do
				if fields[k] and not fields[k].foreign then
					self[k] = tostring(v)
				end
			end
		
			return self
		end;
	}

	return Project
	
现在我们要将example_mixin引入到Project中，这样来操作：

	local Project = Model:extend {
		__tag = ......
		........		........
	
	}:include('mixins.example_mixin')

即是在原来的模型的定义中，extend后面，紧接着跟一个链式操作 :include()，include函数的参数形式如下：

	include (mixin_module, ...)
	
	- mixin_module		可以是mixin_module的模块名，如'mixins.example_mixin'表示引用工程目录下的mixins/目录下的example_mixin.lua文件；这个参量也可以是require 'mixins.example_mixin'后的结果；
	- ...				在引入的时候，附加的一些参数。这些参数将会被传进mixin定义时的那个入口函数。
	
好了，mixin已经被引入到Project模型中了，那么，引入后的效果是什么呢？

## 使用mixin

现在Project中已经引入了example_mixin，这样，每一个Project的实例，现在都有如下5个属性：

1. name;
2. intro;
3. bugs;
4. example_field1;
5. example_field2;

有如下3个方法：
	
1. init;
2. example_method1;
2. example_method2;

这些字段和方法就好像是Project自己原生定义的一样，使用上没有任何区别，按照原来的使用方式用就行了。比如：

	local project = Project(req.params)
	-- 在project实例中调用mixin中的example_method1方法
	project:example_method1()
	....
	project:save()  -- save 存储后，5个字段都会在数据库中有记录

## 修饰器

Mixin中，还可以定义修饰器，即加入__decorators表。比如，我们添加了一个关于save的decorator，整个mixin会是下面这个样子：

	module(..., package.seeall)
	
	return function (...)
		
		......
		
		return {
			__fields = {
				['example_field1'] = {},
				['example_field2'] = {},
			};
			
			__decorators = {
				save = function (osave)
					return function (self, ...)
						if not self then return nil end
						
						.......  		 -- do some actions
						
						return self
					end
				end;
			
			};
			
			init = function (self, t)
				if not t then return self end
				
				example_field1 = t.example_field1
				example_field2 = t.example_field2
				
				return self
			end;
		
			example_method1 = function (param1, param2)
			
				....
			end;
		
			example_method2 = function (param1, param2)
			
				....
			end;
		}
	end
	

## 冲突策略

可是，事情不是那么简单。如果mixin中定义的东西和导入它的模型中定义的内容冲突怎么办？冲突可能发生自如下情况：

1. 字段名的冲突；
2. 普通方法名的冲突；
3. init函数的冲突；
4. decorators的冲突。

Bamboo采用如下策略来解决冲突问题：

1. 字段名的同名冲突，使用mixin中的同名字段及定义覆盖原生的字段定义；
2. 普通方法名的同名冲突，使用mixin中的同名方法覆盖原生的方法；
3. init函数的冲突，先执行完原生模型的init函数后，再执行mixin的init函数；
4. decorators的冲突，mixin中的同名decorator会将原生模型的decorator包裹起来。如果原生模型中没有decorator，则直接将mixin的decorators复制到引用它的那个模型中。


好了。Mixin的内容差不多就这么多。
