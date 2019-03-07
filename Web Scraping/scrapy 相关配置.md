依赖原生正则表达式的程序会更加难以维护！借助一些专门做这些事情的解析库，才能使程序变得清晰。

## 准备

创建虚拟环境：

```powershell
mkvirtualenv --python=C:\Python\Python36\python.exe article
```

安装依赖：

* 安装 `scrapy`

可以使用豆瓣源安装：

```powershell
pip install -i https://pypi.douban.com/simple/ scrapy
```

如果提示 `error: Microsoft Visual C++ 14.0 is required.`，那么独立安装 Twisted 包，即可解决。

可以在 `https://www.lfd.uci.edu/~gohlke/pythonlibs/#twisted` 中找到 python 包。

PS：虚拟环境包：

* `virtualenv`
* `virtualenvwrapper`（或者 `virtualenvwrapper-win`）

***

## 简单的开始

在虚拟环境中：

```cmd
scrapy startproject ArticleSpider
```

便会出现创建成功信息：

```powershell
New Scrapy project 'ArticleSpider', using template directory 'c:\\python\\envs\\article\\lib\\site-packages\\scrapy\\templates\\project', created in:
    C:\GitHub\CrawlerLearning\ArticleSpider

You can start your first spider with:
    cd ArticleSpider
    scrapy genspider example example.com

```

 输入提示的命令：

```powershell
cd ArticleSpider
scrapy genspider jobbole blog.jobbole.com
```

简单的文件结构：

![简单的文件结构](https://wx2.sinaimg.cn/mw690/af9e9c30ly1fw1sk4ap7ij206x0ap3yi.jpg)

