1. 文件的唯一标识 - MD5
   * 假如要处理的重复文件有不同的文件名，最简单的办法就是通过MD5来确定两个文件是不是一样的
2. 保存所有文件标识和路径
   * 接下来遍历所有文件，使用MD5作为key，路径作为value，保存起来
3. 处理重复文件
   * 把上一步建立的字典做一个简单的过滤就能找到重复文件

```python
# _*_coding:utf-8_*_
import hashlib
from pathlib import Path

dup = {}
photo_path = r'C:\Desktop\test\md5test'


def md5sum(filename, blocksize=65536):
    """获得文件的MD5值
    
    Arguments:
        filename {[type]} -- 文件名
    
    Keyword Arguments:
        blocksize {int} -- 可以根据文件大小和CPU性能调整，一般选择的值约等于文件的平均大小。 (default: {65536})
    
    Returns:
        文件的MD5值
    """
    hash = hashlib.md5()
    with open(filename, "rb") as f:
        for block in iter(lambda: f.read(blocksize), b""):
            hash.update(block)
    return hash.hexdigest()


def build_dup_dict(dir_path, pattern='*.*'):
    """保存所有文件标识和路径
    
    Arguments:
        dir_path {[type]} -- [description]
    
    Keyword Arguments:
        pattern {str} -- [description] (default: {'*.*'})
    """
    def save(file):
        hash = md5sum(file)
        if hash not in dup.keys():
            dup[hash] = [file]
        else:
            dup[hash].append(file)

    p = Path(dir_path)
    for item in p.glob('**/' + pattern):
        save(str(item))


def main():
    def get_duplicate():
        return {k: v for k, v in dup.items() if len(v) > 1}

    build_dup_dict(photo_path)
    for hash, files in get_duplicate().items():
        print("{}: {}".format(hash, files))


if __name__ == '__main__':
    main()
```

***

参考：

[使用Python查找目录中的重复文件](https://segmentfault.com/a/1190000013948288)，betacat