Python 中常见的内置数据结构可以统称为容器（container）。

* 数字类型：`int`、`float`、`complex`、`bool`
* 序列类型：`list`、`tuple`、`str`、`range`、`bytes`、`bytearray`
* 映射类型：`dict`
* 集合类型：`set`、`frozenset`

各类型特点：

* 序列类型：有序、可索引、可迭代、可切片
* 映射类型：无序、key唯一、可迭代
* 集合类型：无序、不重复、可迭代

按照是否可变分类如下：

* 可变对象：
  * 序列类型：`list`、`bytearray`
  * 映射类型：`dict`
  * 集合类型：`set`

* 不可变对象：

  * 数字类型：`int`、`float`

  * 序列类型：`tuple`、`bytes`

  * 集合类型：`frozenset`

不可变对象都是可哈希的。

`dict`的`key`或者`set`的值 都必须是可哈希的


此外：

`dict`的查找性能强于`list`，但` dict` 的内存花销大，但是查询速度快。


在`list`中随着`list`数据的增大 查找时间会增大

在`dict`中查找元素不会随着`dict`的增大而增大
