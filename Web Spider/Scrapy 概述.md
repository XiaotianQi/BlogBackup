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

### 创建一个项目

在虚拟环境中：

```cmd
scrapy startproject ArticleSpider
```

便会出现创建成功信息：

```powershell
New Scrapy project 'spider327', using template directory 'c:\\python\\envs\\note\\lib\\site-packages\\scrapy\\templates\\project', created in:
    C:\Desktop\test\spider327

You can start your first spider with:
    cd spider327
    scrapy genspider example example.com

```

### 文件结构

![简单的文件结构](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Web%20Spider/scrapy%20files-1.png)

```text
spider327/
    scrapy.cfg            	# 部署配置文件
    spiders/        	  	# 项目的Python模块,将在这里输入代码
        __init__.py
        items.py          	# 项目的items定义文件
        middlewares.py   	# 中间件
        pipelines.py     	# 管道
        settings.py    	  	# 设置
        spiders/      	    # 存储spider的文件夹
            __init__.py
```

### 创建一个spider

 根据项目创建成功信息，输入提示命令：

```powershell
cd spider327
scrapy genspider jobbole blog.jobbole.com
```

`spider327/spiders/spiders/`文件夹中，目录新增 jobbole.py 文件：

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Web%20Spider/scrapy%20files-2.png)

创建 main.py 文件，便于调试。

```python
# _*_coding:utf-8_*_

import os
import sys
from scrapy.cmdline import execute

sys.path.append(os.path.dirname(os.path.abspath(__file__)))
execute(['scrapy', 'crawl', 'jobbole'])
```

