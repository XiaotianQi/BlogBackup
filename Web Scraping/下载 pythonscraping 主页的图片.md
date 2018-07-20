下载 [pythonscraping](http://www.pythonscraping.com/) 主页的图片。

* 网页 HTML 文件中，SRC 属性中有些是 JS 文件，因此需要剔除；
* 同时，需要剔除的 URL 也包括外链。

```python
if base_url not in url or ".js" in url:
    return None
```

* get_absolute_url() 函数的标准化功能，配合 get_download_paht() 函数，可以创建整齐的下载目录；
* 最后，由 urlretrieve() 函数下载清洗过的 URL。

***

* python 代码如下：

```python
# _*_ coding:utf-8_*_

import os
from urllib.request import urlretrieve, urlopen
from bs4 import BeautifulSoup

download_directory = "downloaded"
base_url = "http://pythonscraping.com"

# 清理和标准化 URL
def get_absolute_url(base_url, source):
    if source.startswith("http://www."):
        url = "http://"+source[11:]
    elif source.startswith("http://"):
        url = source
    elif source.startswith("www."):
        url = "http://"+source
    else:
        url = "{}/{}".format(base_url, source)
    if base_url not in url or ".js" in url:
        return None
    return url
# 建立下载目录
def get_download_path(base_url, absolute_url, download_directory):
    path = absolute_url.replace("www", "")
    path = path.replace(base_url, "")
    path = download_directory+path
    directory = os.path.dirname(path)

    if not os.path.exists(directory):
        os.makedirs(directory)
    return path

html = urlopen("http://www.pythonscraping.com")
bsObj = BeautifulSoup(html, "html.parser")
download_list = bsObj.findAll(src=True)

for download in download_list:
    print("未整理 URL："+download["src"])
    file_url = get_absolute_url(base_url, download["src"])
    if file_url is not None:
        print("已整理并下载 URL："+file_url)
        urlretrieve(file_url, get_download_path(base_url, file_url, download_directory))

```

***

* 输出结果：

可以直观看出，JS 文件已被过滤，以及哪些文件被下载。

```shell
未整理 URL：http://www.pythonscraping.com/misc/jquery.js?v=1.4.4
未整理 URL：http://www.pythonscraping.com/misc/jquery.once.js?v=1.2
未整理 URL：http://www.pythonscraping.com/misc/drupal.js?os2esm
未整理 URL：http://www.pythonscraping.com/sites/all/themes/skeletontheme/js/jquery.mobilemenu.js?os2esm
未整理 URL：http://www.pythonscraping.com/sites/all/modules/google_analytics/googleanalytics.js?os2esm
未整理 URL：http://www.pythonscraping.com/sites/default/files/lrg_0.jpg
已整理并下载 URL：http://pythonscraping.com/sites/default/files/lrg_0.jpg
未整理 URL：http://www.oreilly.com/authors/widgets/669.html
未整理 URL：http://pythonscraping.com/img/lrg%20(1).jpg
已整理并下载 URL：http://pythonscraping.com/img/lrg%20(1).jpg
```