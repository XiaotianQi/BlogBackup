基于爬虫，获取wiki python词条编辑者的 IP 地址，并由 [freegeoip](http://freegeoip.net/json/) 的 API 解析的到编辑者的国籍。  

***

思路：

![思路](https://wx3.sinaimg.cn/mw690/af9e9c30ly1fruqr1emr2j214t08gab5.jpg)

***

python 代码如下：

```python
# _*_coding:utf-8_*_

from urllib.request import urlopen
from urllib.request import HTTPError
from bs4 import BeautifulSoup
import datetime
import random
import re
import json

random.seed(datetime.datetime.now())
# 获取内链
def get_links(article_url):
    html = urlopen("http://en.wikipedia.org" + article_url)
    bsObj = BeautifulSoup(html, "html.parser")
    return bsObj.find("div", {"id":"bodyContent"}).findAll("a",
                        href=re.compile("^(/wiki/)((?!:).)*$"))
# 获取编辑者 IP
def get_history_ip(page_url):
    page_url = page_url.replace("/wiki/", "")
    history_url = "{}{}{}".format("http://en.wikipedia.org/w/index.php?title=",
                                    page_url, "&action=history")
    print("history url is:"+history_url)
    html = urlopen(history_url)
    bsObj = BeautifulSoup(html, "html.parser")
    ip_addresses = bsObj.findAll("a", {"class":"mw-anonuserlink"})
    address_list = set()
    for ip_address in ip_addresses:
        address_list.add(ip_address.get_text())
    return address_list
# 由 freegeoip 的 API解析
def get_country(ip_address):
    try:
        response = urlopen("http://freegeoip.net/json/"+ip_address).read().decode('utf-8')
    except HTTPError:
        return None
    response_json = json.loads(response)
    return response_json["region_name"]

links = get_links("/wiki/Python_(programming_language)")

while (len(links) > 0):
    for link in links:
        print("--------------------")
        history_ips = get_history_ip(link.attrs["href"])
        for history_ip in history_ips:
            country = get_country(history_ip)
            if country is not None:
                print(history_ip+ " is from " + country)
    new_link = links[random.randint(0, len(links)-1).attrs["href"]]
    links = get_links(new_link)
```
***

稍麻烦的地方在：
get_links() 与 get_hitory_ip() 之间的 URL 的传递。

* 首先，get_links() 传入 /wiki/Python_(programming_language)；
* 然后，get_links() 输出 该页爬取的内链列表；
* 接着，get_hitory_ip() 传入 get_links() 刚刚计算的列表；
* 然而，内链连接以 /wiki/ 开头，需要整理成编辑者页面的 URL；
* 因此，需要将传入列表的 URL 进行整理，即 采用 replace() 和 format() 函数。

最后，交于 freegeoip 进行处理，计算出 ip 对应的地区。freegeoip 返回的结果是 JSON，因此需要进行解析。

***

输出结果：

```txt
--------------------
history url is:http://en.wikipedia.org/w/index.php?title=Programming_paradigm&action=history
31.203.136.191 is from Al Asimah
168.216.130.133 is from West Virginia
42.111.56.168 is from National Capital Territory of Delhi
218.17.157.55 is from Guangdong
2605:6000:ec0f:c800:edfd:179f:b648:b4b9 is from Texas
39.36.182.41 is from Punjab
110.55.67.15 is from National Capital Region
223.230.96.108 is from Telangana
92.115.222.143 is from Chișinău Municipality
192.117.105.47 is from Tel Aviv
2a02:c7d:a492:f200:e126:2b36:53ca:513a is from England
223.104.186.241 is from Shandong
193.80.242.220 is from Lower Austria
68.151.180.83 is from Alberta
197.255.127.246 is from Greater Accra Region
--------------------
history url is:http://en.wikipedia.org/w/index.php?title=Object-oriented_programming&action=history
93.136.125.208 is from City of Zagreb
119.152.87.84 is from Punjab
103.16.68.215 is from Karnataka
2605:a601:474:600:2088:fbde:7512:53b2 is from Missouri
103.74.23.139 is from 
81.209.9.65 is from Ostrobothnia
172.56.32.238 is from California
217.225.8.24 is from 
160.53.250.90 is from Geneva
202.70.74.215 is from Central Region
61.0.82.193 is from Uttar Pradesh
47.184.120.251 is from Texas
--------------------
history url is:http://en.wikipedia.org/w/index.php?title=Imperative_programming&action=history
194.181.240.192 is from Lower Silesia
176.60.44.112 is from 
117.136.79.80 is from Guangdong
```

***
参考自《Python 网络数据采集》