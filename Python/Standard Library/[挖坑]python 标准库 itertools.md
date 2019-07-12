Python内置了一个模块itertools，包含了很多函数用于creating iterators for efficient  looping（创建更有效率的循环迭代器），这说明很是霸气，这一小节就来浏览一遍这些函数并留下印象吧，需要这些功能的时候隐约记得这里面有就好。这一小节的内容翻译自itertools模块官方文档。  

## 无限迭代 

```text
count(start, [step])       
从start开始，以后每个元素都加上step。step默认值为1。       
count(10) --> 10 11 12 13 14 ... 

cycle(p)       
迭代至序列p的最后一个元素后，从p的第一个元素重新开始。       
cycle('ABCD') --> A B C D A B C D ... 


repeat(elem [,n])       
将elem重复n次。如果不指定n，则无限重复。       
repeat(10, 3) --> 10 10 10 
```



## 在最短的序列参数终止时停止迭代 

```text
chain(p, q, ...)
迭代至序列p的最后一个元素后，从q的第一个元素开始，直到所有序列终止。       
chain('ABC', 'DEF') --> A B C D E F 

compress(data, selectors)
如果bool(selectors[n])为True，则next()返回data[n]，否则跳过data[n]。       
compress('ABCDEF', [1,0,1,0,1,1]) --> A C E F 

dropwhile(pred, seq)
当pred对seq[n]的调用返回False时才开始迭代。       
dropwhile(lambda x: x<5, [1,4,6,4,1]) --> 6 4 1 

takewhile(pred, seq)
dropwhile的相反版本。       
takewhile(lambda x: x<5, [1,4,6,4,1]) --> 1 4 

ifilter(pred, seq)
内建函数filter的迭代器版本。       
ifilter(lambda x: x%2, range(10)) --> 1 3 5 7 9 

ifilterfalse(pred, seq)
ifilter的相反版本。       
ifilterfalse(lambda x: x%2, range(10)) --> 0 2 4 6 8 

imap(func, p, q, ...)
内建函数map的迭代器版本。       
imap(pow, (2,3,10), (5,2,3)) --> 32 9 1000 

starmap(func, seq)
将seq的每个元素以变长参数(*args)的形式调用func。       
starmap(pow, [(2,5), (3,2), (10,3)]) --> 32 9 1000 

izip(p, q, ...)
内建函数zip的迭代器版本。       
izip('ABCD', 'xy') --> Ax By 

izip_longest(p, q, ..., fillvalue=None)       
izip的取最长序列的版本，短序列将填入fillvalue。       
izip_longest('ABCD', 'xy', fillvalue='-') --> Ax By C- D- 

tee(it, n)
返回n个迭代器it的复制迭代器。 

groupby(iterable[, keyfunc])
这个函数功能类似于SQL的分组。使用groupby前，首先需要使用相同的keyfunc对iterable进行排序，比如调用内建的sorted函数。然后，groupby返回迭代器，每次迭代的元素是元组(key值,  iterable中具有相同key值的元素的集合的子迭代器)。或许看看Python的排序指南对理解这个函数有帮助。       
groupby([0, 0, 0, 1, 1, 1, 2, 2, 2]) --> (0, (0 0 0)) (1, (1 1 1)) (2, (2 2 2)) 
```



## 组合迭代器 

```text
product(p, q, ... [repeat=1])
笛卡尔积。       
product('ABCD', repeat=2) --> AA AB AC AD BA BB BC BD CA CB CC CD DA DB DC DD 

permutations(p[, r])
去除重复的元素。       
permutations('ABCD', 2) --> AB AC AD BA BC BD CA CB CD DA DB DC 

combinations(p, r)       
排序后去除重复的元素。       
combinations('ABCD', 2) --> AB AC AD BC BD CD 

combinations_with_replacement()
排序后，包含重复元素。       
combinations_with_replacement('ABCD', 2) --> AA AB AC AD BB BC BD CC CD DD 
```





***

参考：

https://www.cnblogs.com/huxi/archive/2011/07/01/2095931.html