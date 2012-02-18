# QuerySet

QuerySet 是从数据库中检索出来的模型实例列表。

## 生成QuerySet
目前Bamboo中可以有如下API产生QuerySet，

- `SomeModel:all(is_rev)`	

- `SomeModel:slice(start, stop, is_rev)`

- `SomeModel:filter(query_args, is_rev, starti, length, dir)`

## QuerySet的API
有如下API可供QuerySet使用：

- `query_set:filter(query_args, is_rev, starti, length, dir)`

	在query_set中依照传入的query_args及附加参数（继续）进行过滤。由于filter函数本身也返回query_set，因此，可以写出级联表达式来。

- `query_set:sortBy(field, direction, sort_func, ...)`

	在query_set中按传入参数进入排序。
	- field	排序依照的字段
	- direction	排序的方向，可取 'asc' 或 'dsc'
	- sort_func	排序时使用的函数（不写的话，使用内置默认函数）
	- ...	还可以是一组 filed, direction, sort_func，可以进行二阶排序

- `query_set:querySetIds()`

	生成query_set的id列表。

- `query_set:del()`

	删除此query_set中的实例对象。

- `query_set:fakeDel()`

	删除此query_set中的实例对象（假删）。

- `query_set:trueDel()`

	删除此query_set中的实例对象（真删）。

## QuerySet的继承关系图

![QuerySet继承关系图](/daogangtang/bamboo_doc_cn/blob/master/images/query_set.png)