URLconf 全称 URL configuration。这个模块包含包含 URL 模式（简单的正则表达式）到 views（视图）的简单映射。根据用户或者浏览器发起的 URL 请求，调用正确的 Django 视图，并从 URL 中提取参数。

***

### **命名组**

```python
urlpatterns = [
    url(r'^articles/(?P<year>[0-9]{4})/$', views.archive_year),
    url(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]+)/$', views.archive_month),
    url(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]+)/(?P<day>[0-9]+)/$', views.article_day),
]
```

* 不需要添加一个前导的反斜杠，Django 在检查 URL 模式前，会移除每一个申请的 URL 开头的斜杠 /。
* 在 Python 正则表达式中，命名正则表达式组的语法是 (?P\<name>pattern)，其中 name 是组的名称，pattern 是要匹配的模式。由此，捕获的值作为关键字参数而不是位置参数传递给视图函数。
* 如果存在命名参数，则使用命名参数，忽略非命名参数。
* 捕获的参数总是字符串。即使 [0-9]{4} 是整数，但是传递给 views.archive_year 亦是字符串。

***

### **包含其他URLconfs**

```python

extra_patterns = [
    url(r'^(?P<year>[0-9]{4})/$', views.archive_year),
    url(r'^(?P<year>[0-9]{4})/(?P<month>[0-9]+)/$', views.archive_month),
    url(r'^(?P<year>[0-9]{4})/(?P<month>[0-9]+)/(?P<day>[0-9]+)/$', views.article_day),
]

urlpatterns = [
    ...
    url(r'^articles/', include(extra_patterns),
    ...
]
```

* 实际上将一部分 URL 放置于其它 URL 下面，由 extra_patterns 中的视图函数处理。可以用于移除 URL 配置中重复的部分。
* 例子中的正则表达式没有包含 $，但是包含一个末尾的 /。 处理时去掉URL 中匹配的部分并将剩下的字符串发送给包含的 URLconf 做进一步处理。

也使用一个 url() 实例的列表，达到同样的目的。

```python
urlpatterns = [
    url(r'^articles/', include([
        url(r'^(?P<year>[0-9]{4})/$', views.archive_year),
        url(r'^(?P<year>[0-9]{4})/(?P<month>[0-9]+)/$', views.archive_month),
        url(r'^(?P<year>[0-9]{4})/(?P<month>[0-9]+)/(?P<day>[0-9]+)/$', views.article_day),
    ])),
]
```

***

### **URL的反向解析**

当改进 URL 设计之后无需在项目源码中大范围搜索、替换失效的 URL。此时，URL 一定不能硬编码。根据 DRY 原则，Django 只需在 URL 映射中设计 URL。即是根据 Django 视图的标识和将要传递给它的参数的值，获取与之关联的 URL。需要使用命名的 URL 模式

在需要URL 的地方，对于不同层级，Django 提供不同的工具用于URL 反查：

> * 在模板中：使用 url 模板标签。
> * 在 Python 代码中：使用 reverse() 函数。
> * 在更高层的与处理 Django 模型实例相关的代码中：使用 get\_absolute_url() 方法。

```python
urlpatterns = [
    url(r'^articles/(?P<year>[0-9]{4})/$', views.archive_year, name = 'archive_year'),
]
```
