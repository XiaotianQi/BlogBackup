Scrapy settings 允许自定义所有Scrapy组件的行为，包括core, extensions, pipelines and spiders themselves.

设置的基础结构提供了一个键值映射的全局名称空间，代码可以使用该名称空间来提取配置值。可以通过不同的机制填充设置，如下所述。

可以使用不同的机制来填充设置，每种机制都有不同的优先级。 以下是按优先级降序排列的列表：

1. Command line options（最优先）
2. Settings per-spider
3. 项目settings模块
4. 每个命令的默认设置
5.  默认的全局设置（优先级最低）

设置scrapy路径：

```python
# settings.py

BASE_DIR = os.path.dirname(os.path.abspath(os.path.dirname(__file__)))
sys.path.insert(0, os.path.join(BASE_DIR, 'NewsSpider'))
```

## 获取 settings.py 中的值

* `.from_crawler(cls, settings)`



* `.from_settings(cls, settings)`



* `from xx.settings import xxx`



## 内置设置参考