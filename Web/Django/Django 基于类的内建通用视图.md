在视图开发过程中发现的某些共同的用法和模式，然后将之抽象出来，以便能够写更少的代码快速的实现常见的视图。这些抽象出来的是 Django 通用视图。

通用视图虽然能够提升开发速度，但是也有一些限制。在项目中，总会遇到通用视图无法满足需求的时候。此时子类化通用视图，并且重写它们的属性或者方法，能够扩展通用视图。

***

### **博客代码示例**

以搭建博客的涉及的 models 和 views 部分代码，体现出重写通用视图能够拓展功能，并简洁代码使之简单易懂。

* models.py

```python
class Category(models.Model):
    name = models.CharField(max_length=100)

    def __str__(self):
        return self.name


class Tag(models.Model):
    name = models.CharField(max_length=100)

    def __str__(self):
        return self.name


class Post(models.Model):
    title = models.CharField(max_length=70)
    body = models.TextField()
    created_time = models.DateTimeField(auto_now_add=True)
    modified_time = models.DateTimeField(auto_now=True)
    excerpt = models.CharField(max_length=200, blank=True)

    category = models.ForeignKey(Category)
    tag = models.ManyToManyField(Tag, blank=True)
    author = models.ForeignKey(User)
    # 记录阅读量
    views = models.PositiveIntegerField(default=0)

    def increase_views(self):
        self.views += 1
        self.save(update_fields=['views'])

    def get_absolute_url(self):
        return reverse('blog:detail', kwargs={'pk': self.pk})

    def __str__(self):
        return self.title

    class Meta:
        ordering = ['-created_time']
```

* views.py

```python
class IndexView(ListView):
    model = Post
    template_name = 'blog/index.html'
    context_object_name = 'post_list'
    paginate_by = 7


class CategoryView(ListView):
    model = Post
    template_name = 'blog/index.html'
    context_object_name = 'post_list'

    def get_queryset(self):
        cate = get_object_or_404(Category, pk=self.kwargs.get('pk'))
        return super(CategoryView, self).get_queryset().filter(category=cate)


class PostDetailView(DetailView):
    model = Post
    template_name = 'blog/detail.html'
    context_object_name = 'post'

    def get(self, request, *args, **kwargs):
        response = super(PostDetailView, self).get(request, *args, **kwargs)
        self.object.increase_views()
        return response

    def get_object(self, queryset=None):
        post = super(PostDetailView, self).get_object(queryset=None)
        md = markdown.Markdown(extensions=[
            'markdown.extensions.extra',
            'markdown.extensions.codehilite',
            TocExtension(slugify=slugify),
        ])
        post.body = md.convert(post.body)
        post.toc = md.toc
        return post

    def get_context_data(self, **kwargs):
        context = super(PostDetailView, self).get_context_data(**kwargs)
        form = CommentForm()
        comment_list = self.object.comment_set.all()
        context.update({
            'form': form,
            'comment_list': comment_list
        })
        return context
```

然后，将视图解析到 url，并在 templates 中渲染，不再赘述。

***

### **友好支持 templates**

get_context_data。

通用视图 ListView 的 context_object_name 属性指定要使用的上下文变量，这样便于设计 templates 时，易于理解。

例如，在 views 中，视图类加入了 *context_object_name = 'post_list'*，则在 设计 templates 时，便宜查找和编写。

```HTML
{% block main %}
    {% for post in post_list %}
    ...
    ...
    ...
    {% endfor %}
{% endblock main %}
```

***

### **添加额外的上下文**

get_context_data。

通用视图 DetailView 的 在get_context_data 属性，默认的实现只是简单地给模板添加要展示的对象，但是可以覆盖它来展示更多信息：

```python
def get_context_data(self, **kwargs):
        context = super(PostDetailView, self).get_context_data(**kwargs)
        form = CommentForm()
        comment_list = self.object.comment_set.all()
        context.update({
            'form': form,
            'comment_list': comment_list
        })
        return context
```

上文这段代码，便是添加了评论功能。

***

### **动态过滤**

get_queryset()。

通用视图 ListView 的 get_queryset()，在给定的列表页面中根据 URL 中的关键字来过滤对象。这样在浏览器中点击一个博文类别，可以看到全部属于这个类别的博文。

让这种方式能够工作的关键点，在于当类视图被调用时，各种有用的对象被存储在self上；同request(self.request)一样，其中包含了从URLconf中获取到的位置参数 (self.args)和基于名字的参数(self.kwargs)(关键字参数)。

```python
def get_queryset(self):
    cate = get_object_or_404(Category, pk=self.kwargs.get('pk'))
    return super(CategoryView, self).get_queryset().filter(category=cate)
```