Hash，译做“散列”，也有直接音译为“哈希”的。把任意长度的输入，通过某种hash算法，变换成固定长度的输出，该输出就是散列值，也称摘要值。该算法就是哈希函数，也称摘要函数。

通过摘要函数对任意长度的数据计算出固定长度的摘要`digest`，目的是为了提供一个验证文件未被篡改的方法。摘要函数是一个单向函数，计算`digest`很容易，但通过`digest`反推原始数据却非常困难。而且，最关键的是，对原始数据做一个比特位的修改，都会导致计算出的摘要完全不同。

有没有可能两个不同的原始数据通过某个摘要算法会得到相同的摘要？完全有可能，因为任何摘要算法都是把无限多的数据集合映射到一个有限的集合中，这种转换是一种压缩映射，也就是说，散列值的空间通常远小于输入的空间，不同的输入可能会散列成相同的输出，所以不可能从散列值来唯一的确定输入值。

那你上面说的啥？自相矛盾吗？虽然摘要值有可能相同，但是现在广泛应用的摘要算法是在一个非常巨大的样本空间中计算的，摘要值通常都有好几百位，在使用过程中碰到两个摘要完全相同的概率非常非常非常小，可以忽略不计。

要注意，摘要算法不是加密算法，不能用于加密数据（因为无法通过摘要反推明文），只能用于防止数据被篡改，但是它的单向计算特性决定了可以在不存储明文口令的情况下验证用户口令。

`MD5`是最常见的摘要算法，速度很快，生成结果是固定的16字节，通常用一个32位的16进制字符串表示。SHA1算法更安全点，它的结果是20字节长度，通常用一个40位的16进制字符串表示。而比SHA1更安全的算法是SHA256和SHA512等等，不过越安全的算法越慢，并且摘要长度更长。

## hashlib模块

Python内置的hashlib模块为我们提供了多种安全方便的摘要方法。

在大部分操作系统下，hashlib模块支持`md5()`,`sha1()`, `sha224()`, `sha256()`, `sha384()`, `sha512()`, `blake2b()`，`blake2s()`，`sha3_224()`, `sha3_256()`, `sha3_384()`, `sha3_512()`, `shake_128()`, `shake_256()`等多种hash构造方法。这些构造方法在使用上通用，返回带有同样接口的hash对象，对算法的选择，差别只在于构造方法的选择。例如sha1()能创建一个SHA-1对象，sha256()能创建一个SHA-256对象。然后就可以使用通用的`update()`方法将bytes类型的数据添加到对象里，最后通过`digest()`或者`hexdigest()`方法获得当前的摘要。`update()`方法现在只接受bytes类型的数据，不接收str类型。

## 基本流程

获得bytes类型字符串`b'Nobody inspects the spammish repetition'`的摘要的过程：

```python
In [2]: m = hashlib.sha256()			# 通过构造函数获得一个hash对象

In [3]: m.update(b"Nobody inspects")

In [4]: m.update(b" the spammish repetition")

In [5]: m.digest()						# 获得bytes类型消息摘要
Out[5]: b'\x03\x1e\xdd}Ae\x15\x93\xc5\xfe\\\x00o\xa5u+7\xfd\xdf\xf7\xbcN\x84:\xa6\xaf\x0c\x95\x0fK\x94\x06'

In [6]: m.hexdigest()					# 获得16进制str类型的消息摘要
Out[6]: '031edd7d41651593c5fe5c006fa5752b37fddff7bc4e843aa6af0c950f4b9406'

In [7]: m.digest_size					# 查看消息摘要的位长
Out[7]: 32

In [8]: m.block_size					# 查看消息摘要的内部块大小
Out[8]: 64

# 更简洁的用法
In [9]: hashlib.sha256(b"Nobody inspects the spammish repetition").hexdigest()
Out[9]: '031edd7d41651593c5fe5c006fa5752b37fddff7bc4e843aa6af0c950f4b9406'
```

`hashlib.new(name[, data])`

一个通用的构造方法，name是某个算法的字符串名称，data是可选的bytes类型待摘要的数据。不过，此方法运算速度相对以上方法较慢。

```python
In [10]: h = hashlib.new('sha256', b"abc")

In [11]: h.digest()
Out[11]: b'\xbax\x16\xbf\x8f\x01\xcf\xeaAA@\xde]\xae"#\xb0\x03a\xa3\x96\x17z\x9c\xb4\x10\xffa\xf2\x00\x15\xad'
```

## hashlib模块的两个常量属性

`hashlib.algorithms_guaranteed`

所有平台中，模块支持的hash算法列表

`hashlib.algorithms_available`

当前Python解释器环境中，模块支持的hash算法列表

## hash对象的两个常量属性

`hash.digest_size`

hash结果的长度

`hash.block_size`

hash内部块的大小

## hash对象的属性

`hash.name`

hash算法名称字符串

## hash对象的方法

`hash.update(arg)`

更新hash对象。连续的调用该方法相当于连续的追加更新。例如m.update(a); m.update(b)相当于m.update(a+b)。注意，当数据规模较大的时候，Python的GIL在此时会解锁，用于提高计算速度。

一定要理解update()的作用，由于消息摘要是只针对当前状态产生的，所以每一次update后，再次计算hexdigest()的值都会不一样。

`hash.digest()`

返回bytes格式的消息摘要

`hash.hexdigest()`

与digest方法类似，不过返回的是两倍长度的字符串对象，所有的字符都是十六进制的数字。通常用于邮件传输或非二进制环境中。通常我们比较摘要时，比较的就是这个值！

`hash.copy()`

返回一个hash对象的拷贝

## 使用场景

最常用的就是密码加密！密码加密不像数据加密，通常不需要反向解析出明文。

以上加密算法虽然很厉害，但仍然存在缺陷，通过撞库可以反解。所以必要对加密算法中添加自定义key再来做加密。

**加盐**：额外给原始数据添加一点自定义的数据，使得生成的消息摘要不同于普通方式计算的摘要。

我们一般认为用户名是不能重复的。将用户名作为盐的一部分，就不会出现一样的密码消息摘要了。

```python
def create_md5(pwd, salt):
    md5_obj = hashlib.md5()
    md5_obj.update((pwd+salt).encode('utf8'))
    return md5_obj.hexdigest()
```

***

参考：

http://www.liujiangblog.com/course/python/58