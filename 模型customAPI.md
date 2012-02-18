# 模型custom API

Redis相对于传统的SQL数据库，一个很大的优势就是灵活性。为了能利用这种灵活性，Bamboo设计了一套完整的的自定义k-v存储接口，用户可以方便地使用它们。它们特别适用于记录一些模型相关的属性到数据库中。 

## APIs
#### SomeModel:setCustom(key, val, st, scores)	

	创建一个custom key，将val值写入此key中。val可以为string, list, st只能取nil, 'string', 'list', 'set', 'zset', 'hash' 中的一个

#### SomeModel:getCustom(key, atype)	

	获取custom key的所有内容

#### SomeModel:delCustom(key)	

	删除此custom key
	
#### SomeModel:updateCustom(key, val)	

	将val值更新到custom key中去，注意，是覆盖关系

#### SomeModel:removeCustomMember(key, val)	

	删除custom key中的val元素

#### SomeModel:addCustomMember(key, val, score)	

	添加一个member到custom key中去

#### SomeModel:hasCustomMember(key, mem)

	判断custom key中，是否有成员 mem。
	

#### SomeModel:numCustom(key)	

	测量custom key的值的长度

## 说明
在内部，custom key是被限制在SomeModel下面的。也就是说，不存在独立的custom key，总是需要依附某一个model而存在。

比如，User模型，使用  

	User:setCustom('test', 'have a test')  

后，在redis中存储的key是 `User:custom:test`, 值为 'have a test' 。

如果实在找不到要用到的custom key与哪一个模型有关联，就用`Model`模型吧。

执行  

	Model:setCustom('test', 'have a test')  

后，在redis中存储的key是 `Model:custom:test`, 值为 'have a test' 。 


## Note

1. 虽然说custom支持string, list, set, zset四种存储结构，但每种结构最基本的单元还是一个字符串，在设计的时候要考虑。
2. 尽量不要使用纯数字作为key参数，名字尽量取得有意义；
3. 将custom key限制在某一个模型名字空间下面，是为了防止滥用custom key。
