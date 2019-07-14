简要记录 Python 迈入门槛后，第一个学习的项目，并基于 Django 搭建这个简单不如说简陋的 Blog 网站。网站仍有许多问题亟待结果，下一刻的事情就放到下一个解决，现在将 Django 一些简明扼要的知识点总结一下。

***

## 目录结构

### Project 目录结构

进入终端

```shell
django-admin startproject blogproject
```

blogproject 项目目录结构

```shell
blogproject/
    manage.py
    blogproject/
        __init__.py
        settings.py
        urls.py
        wsgi.py
```

这些目录和文件的用处:

* 最外层 blogproject/ 根目录是项目容器
* manage.py 项目管理器，与项目进行交互的命令行工具集入口。可执行python manage.py 查看所有命令。
* 里面一层的 blogproject/ 目录就是你项目的实际 Python 包，引用它内部任何东西时需要用到的 Python 包名。
* blogproject/\_\_init\_\_.py ：一个用于指明此目录是 Python 包的空白文件。
* blogproject/settings.py ：项目的总配置文件。包含数据库、 Web 应用、时间等各种配置。
* blogproject/urls.py ：url 映射配置文件。可视其为网站的目录。
* blogproject/wsgi.py ：Python 应用程序或框架与web服务器之间的接口。全称是 Python Web Server Gateway Interface。

***

### App 目录结构

在 Django 中，每一个应用都是一个 Python 包。在每个应用目录中，都可以发现 \_\_init\_\_.py 文件。
> 项目和应用有什么区别？  
>应用是一个专门做某件事的网络应用程序，比如博客系统、公共记录的数据库、或者简单的投票程序。项目则是一个网站使用的配置和应用的集合。项目可以包含很多个应用；而应用可以被很多个项目使用。

根目录下

```shell
python manage.py startapp blog
```

blog 应用目录结构

```shell
blog/
    __init__.py
    admin.py
    apps.py
    models.py
    tests.py
    urls.py
    views.py
```

* admin.py : 后台管理系统配置文件。
* apps.py : 应用的一些配置
* models.py : 使用 ORM 框架，Object Relational Mapping(对象关系映射)。主要采用 Python 类来描述数据表。基于这个类，可以使用简单的 Python 的代码来创建、检索、更新、删除 数据库中的记录，而无需写一条又一条的SQL语句。
* test.py : 自动化测试模块，在这里编写测试脚本。
* urls.py ： 建立url与响应函数之间的关系。
* views.py ： 执行响应的代码模块，代码逻辑处理的主要地点。响应用户 http 请求，进行逻辑处理，返回 html 页面。

***

### Project 和 App 之间区别

* Project 的作用是提供配置文件，例如 App 安装列表，数据库连接信息、url 配置、Template 配置等。
* 一个 App 是一套 Django 功能的集合，通常包括模型和视图，并以 Python 的包结构的方式存在。
* 一个 Project 可以包含很多个 App 以及对它们的配置。
* App 的关键点是它们是很容易移植到其他 Project 或被多个 Project 复用。

***

## Django 简洁工作流程

Django 简单易懂工作流程

1. 用户通过浏览器请求一个页面
1. 通过 urls.py 文件和请求的 URL 找到相应的 Views
1. 调用 Views 中的函数
1. Views 通过操控 Models 和 Template 得到 Response
1. Response返回到浏览器，呈现用户

具体流程图如下：

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Django/Django%20data%20flow.png)

```text
1. 用户通过浏览器请求一个页面
2. 请求到达Request Middlewares，中间件对request做一些预处理或者直接response请求
3. URLConf通过urls.py文件和请求的URL找到相应的View
4. View Middlewares被访问，它同样可以对request做一些处理或者直接返回response
5. 调用View中的函数
6. View中的方法可以选择性的通过Models访问底层的数据
7. 所有的Model-to-DB的交互都是通过manager完成的
8. 如果需要，Views可以使用一个特殊的Context
9. Context被传给Template用来生成页面
a. Template使用Filters和Tags去渲染输出
b. 输出被返回到View
c. HTTPResponse被发送到Response Middlewares
d. 任何Response Middlewares都可以丰富response或者返回一个完全不同的response
e. Response返回到浏览器，呈现给用户
```

***

### Middleware

在Django中，Middleware可以渗入处理流程的四个阶段：request，view，response和exception。相应的，在每个Middleware类中都有rocess_request，process_view，process_response 和 process_exception这四个方法。可以定义其中任意一个或者多个方法，这取决于希望该Middleware作用于哪个阶段。每个方法都可以直接返回response对象。

Middleware是在Django BaseHandler的load_middleware方法执行时加载的，加载之后会建立四个列表作为处理器的实例变量：

```text\
_request_middleware：process_request方法的列表
_view_middleware：process_view方法的列表
_response_middleware：process_response方法的列表
_exception_middleware：process_exception方法的列表
```

中间件出现的顺序非常重要：在request和view的处理阶段，Django按照MIDDLEWARE_CLASSES中出现的顺序来应用中间件，而在response和exception异常处理阶段，Django则按逆序来调用它们。也就是说，Django将MIDDLEWARE_CLASSES视为view函数外层的顺序包装子：在request阶段按顺序从上到下穿过，而在response则反过来。以下两张图可以更好地帮助你理解：

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Django/Django%20middleware.png)

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Django/Django%20middleware%201.png)

### URLConf(URL映射)

如果处理request的中间件都没有直接返回response，那么Django会去解析用户请求的URL。URLconf就是Django所支撑网站的目录。它的本质是URL模式以及要为该URL模式调用的视图函数之间的映射表。通过这种方式可以告诉Django，对于这个URL调用这段代码，对于那个URL调用那段代码。具体的，在Django项目的配置文件中有ROOT_URLCONF常量，这个常量加上根目录"/"，作为参数来创建django.core.urlresolvers.RegexURLResolver的实例，然后通过它的resolve方法解析用户请求的URL，找到第一个匹配的view。

***

## MVC 设计模式

MVC 设计模式是把数据存取逻辑、业务逻辑和表现逻辑组合在一起的概念。这种设计模式关键的优势在于各种组件都是**松散结合**。

* Model（模型）是应用程序中用于处理应用程序**数据逻辑**的部分。通常模型对象负责在数据库中存取数据。
* View（视图）是应用程序中处理**数据显示**的部分。选择显示哪些数据要显示以及怎样显示的部分，由视图和模板处理。
* Controller（控制器）是应用程序中处理**用户交互**的部分。根据用户输入委派视图的部分，由 Django 框架根据 URLconf 设置，对给定 URL 调用适当的 Python 函数。通常控制器负责从视图读取数据，控制用户输入，并向模型发送数据。

MVC是三个单词的首字母缩写，它们是Model（模型）、View（视图）和Controller（控制）。

这个模式认为，程序不论简单或复杂，从结构上看，都可以分成三层。

> 1. 最上面的一层，是直接面向最终用户的"视图层"（View）。它是提供给用户的操作界面，是程序的外壳。
> 2. 最底下的一层，是核心的"数据层"（Model），也就是程序需要操作的数据或信息。
> 3. 中间的一层，就是"控制层"（Controller），它负责根据用户从"视图层"输入的指令，选取"数据层"中的数据，然后对其进行相应的操作，产生最终结果。

这三层是紧密联系在一起的，但又是互相独立的，每一层内部的变化不影响其他层。每一层都对外提供接口（Interface），供上面一层调用。这样一来，软件就可以实现模块化，修改外观或者变更数据都不用修改其他层，大大方便了维护和升级。

***

## MTV 设计模式

Django 有意弱化 Controller ,更注重 Template（模板）。这便是 Model 、Template 和 Views，即 MTV 设计模式。

* Template（模板）把数据渲染变成可浏览的网页。

***

参考：

[谈谈MVC模式](http://www.ruanyifeng.com/blog/2007/11/mvc.html)，阮一峰

[[Django运行方式及处理流程总结](https://segmentfault.com/a/1190000002399134)，WuXianglong                                                                                         