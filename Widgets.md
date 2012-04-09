# Widgets

Widgets是Bamboo为提升写html的效率而封装的页面控件。

目前，有如下几个widgets：

- text;
- checkbox;
- radio;
- select;


### Text 控件

用法：

	{< text name="xxx", value=foobar, class="xxxx" >}

可以添加的参数有：

	- name			name属性；
	- class			class属性；
	- id			id属性；
	- value			text的值，是一个字符串


### Checkbox 控件

用法：

	{< checkbox name="xxx", value=foobar, class="xxxx" >}

可以添加的参数有：

	- name			name属性；
	- class			class属性；
	- id			id属性；
	- value_field	指定从控件的value参数中充当option的val的字段名称
	- caption_field	指定从控件的value参数中充当option的caption的字段名称
	- value			此控件的值，是一个列表。
		如果没有指定value_field和caption_field，则这个列表中的每个元素是下面这个样子
		{ option_val, option_caption }
		如果指定了value_field和caption_field，则这个列表中的每个元素是下面这个样子
		{ [value_field] = option_val, [caption_field] = option_caption }
	- checked		用于指定哪些被选中。为一个string或string list，值就是若干option_val


### Radio 控件

用法：

	{< radio name="xxx", value=foobar, class="xxxx" >}

可以添加的参数有：

	- name			name属性；
	- class			class属性；
	- id			id属性；
	- value_field	指定从控件的value参数中充当option的val的字段名称
	- caption_field	指定从控件的value参数中充当option的caption的字段名称
	- value			此控件的值，是一个列表。
		如果没有指定value_field和caption_field，则这个列表中的每个元素是下面这个样子
		{ option_val, option_caption }
		如果指定了value_field和caption_field，则这个列表中的每个元素是下面这个样子
		{ [value_field] = option_val, [caption_field] = option_caption }
	- checked		用于指定哪些被选中。为一个string或string list，值就是若干option_val
	- layout		如果取值'vertical'，则竖排显示


### Select 控件 

用法：

	{< select name="xxx", value=foobar, class="xxxx" >}

可以添加的参数有：

	- name			name属性；
	- class			class属性；
	- id			id属性；
	- value_field	指定从控件的value参数中充当option的val的字段名称
	- caption_field	指定从控件的value参数中充当option的caption的字段名称
	- value			此控件的值，是一个列表。
		如果没有指定value_field和caption_field，则这个列表中的每个元素是下面这个样子
		{ option_val, option_caption }
		如果指定了value_field和caption_field，则这个列表中的每个元素是下面这个样子
		{ [value_field] = option_val, [caption_field] = option_caption }
	- selected		用于指定哪些被选中。为一个string或string list，值就是若干option_val
	

