## `dict`

`dict` 属于 `mapping` 类型

`key` 必须是不可变对象。

> A dictionary’s keys are *almost* arbitrary values.  Values that are not hashable, that is, values containing lists, dictionaries or other mutable types (that are compared by value rather than by object identity) may not be used as keys.  Numeric types used for keys obey the normal rules for numeric comparison: if two numbers compare equal (such as `1` and `1.0`) then they can be used interchangeably to index the same dictionary entry.  (Note however, that since computers store floating-point numbers as approximations it is usually unwise to use them as dictionary keys.)

