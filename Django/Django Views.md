基于函数的视图的问题在于，虽然它们很好地覆盖了简单的情形，但是不能扩展或自定义它们，即使是一些简单的配置选项，这让它们在现实应用中受到很多限制。类的可操作性要强。

Django 的 URL 解析器希望将请求和关联的参数发送到可调用函数，而不是类。不过，基于类的视图具有一个 as_view() 类方法，它返回一个可以在请求时调用的函数到达与相关模式匹配的 URL。 该函数创建一个类的实例并调用其 dispatch() 方法。此方法的返回值与基于函数的视图的返回值完全相同，即 HttpResponse 的某种形式。

***

### **Mixin**

Django 使用基类和 Mixin 来构建基于类的通用视图。

Mixin 是多继承的一种形式，可以将来自多个父类的行为和属性组合在一起。

继承强调的是 I AM，而 Mixin 着重于 I CAN。

***

### **基于类的视图**

一个基本的博客文章分类视图函数：

* views.py

```python
def category(request, pk):
    cate = get_object_or_404(Category, pk=pk)
    post_list = Post.objects.filter(category=cate)
    return render(request, 'blog/index.html', context={'post_list': post_list})
```

与之类似并基于类的视图：

* views.py

```python
class CategoryView(ListView):
    model = Post
    template_name = 'blog/index.html'
    context_object_name = 'post_list'

    def get_queryset(self):
        cate = get_object_or_404(Category, pk=self.kwargs.get('pk'))
        return super(CategoryView, self).get_queryset().filter(category=cate)
```

其中，model = Post 可以使用 queryset 参数来指定一个对象列表：

```python
queryset = Post.objects.all()
```

由此，可以实现排序、过滤等功能：

```python
queryset = Post.objects.order_by('name')
```

```python
queryset = Post.objects.order_by(name='Django')
```

***

### **通用视图**

所有的视图类继承自 View 类，它负责将视图连接到 URL、HTTP 方法调度和其它简单的功能。使用通用视图可以极大的提高开发速度，遇到通用视图无法满足需求的时候,子类化通用视图，并且重写属性或者方法。

***

### **例1：记录文章阅读量**

实现此项功能，可以在 model 中实现，也可以在 views 中实现。

在 model 中，如下：

* models.py

```python
class Post(models.Model):
    ...
    # 记录阅读量
    views = models.PositiveIntegerField(default=0)

    def increase_views(self):
        self.views += 1
        self.save(update_fields=['views'])
    ...
```

在 views 中，如下：

当然需要先在 models 进行声明：

* models.py

```python
class Post(models.Model):
    ...
    # 记录阅读量
    views = models.PositiveIntegerField(default=0)
    ...
```

然后在 views 中：

* views.py

```python

class PostDetailView(DetailView):

    queryset = Post.objects.all()

    def get_object(self):
        object = super(PostDetailView, self).get_object()
        object.views += 1
        object.save()
        return object
```

***

### **例2：基于通用视图的表单处理**

表单处理的3条常见分支：

>* 初始的GET （空白或预填充的表单）
>* 带有非法数据的POST（通常重新显示表单和错误信息）
>* 带有合法数据的POST（处理数据并重定向）

* views.py

```python
def get_name(request):
    if request.method == 'POST':
        form = NameForm(request.POST)
        if form.is_valid():
            return HttpResponseRedirect('/thanks/')
    else:
        form = NameForm()

    return render(request, 'name.html', {'form': form})
```

未使用通用视图，实现这些功能经常导致许多重复的样本代码。

以一个基本联系人表单为例：

* forms.py

```python
class ContactForm(forms.Form):
    name = forms.CharField()
    message = forms.CharField(widget=forms.Textarea)

    def send_email(self):
        pass
```

* views.py

```python
class ContactView(FormView):
    template_name = 'contact.html'
    form_class = ContactForm
    success_url = '/thanks/'

    def form_valid(self, form):
        form.send_email()
        return super(ContactView, self).form_valid(form)
```