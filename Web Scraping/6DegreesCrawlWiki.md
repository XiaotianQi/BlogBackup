你和任何一个陌生人之间所间隔的人不会超过六个，也就是说，最多通过五个中间人你就能够认识任何一个陌生人，见图所示。这就是六度分割理论，也叫小世界理论。

![数据库格式](https://wx4.sinaimg.cn/mw690/af9e9c30ly1fu4vivb0r6j20pj0cuq3a.jpg)

所示代码由 `Kevin_Bacon` 开始。

```python
# _*_coding:utf-8_*_

from urllib.request import urlopen
from bs4 import BeautifulSoup
import re
import pymysql

conn = pymysql.connect(
    host='127.0.0.1',
    port=3306,
    user='root',
    password='qixt',
    db='mysql',
    charset='utf8'
)
cur = conn.cursor()
cur.execute("USE wikipedia")

def pageScraped(url):
    cur.execute("SELECT * FROM pages WHERE url = %s", (url))
    if cur.rowcount == 0:
        return False
    page = cur.fetchone()

    cur.execute("SELECT * FROM links WHERE fromPageId = %s", (int(page[0])))
    if cur.rowcount == 0:
        return False
    return True

def insertPageIfNotExists(url):
    cur.execute("SELECT * FROM pages WHERE url = %s", (url))
    if cur.rowcount == 0:
        cur.execute("INSERT INTO pages (url) VALUES (%s)", (url))
        conn.commit()
        return cur.lastrowid
    else:
        return cur.fetchone()[0]

def insertLink(fromPageId, toPageId):
    cur.execute("SELECT * FROM links WHERE fromPageId = %s AND toPageId = %s", (int(fromPageId), int(toPageId)))
    if cur.rowcount == 0:
        cur.execute("INSERT INTO links (fromPageId, toPageId) VALUES (%s, %s)", (int(fromPageId), int(toPageId)))
        conn.commit()

def getLinks(pageUrl, recursionLevel):
    if recursionLevel > 4:
        return
    pageId = insertPageIfNotExists(pageUrl)
    html = urlopen("http://en.wikipedia.org"+pageUrl)
    bsObj = BeautifulSoup(html, "lxml")
    for link in bsObj.findAll("a", href=re.compile("^(/wiki/)((?!:).)*$")):
        insertLink(pageId, insertPageIfNotExists(link.attrs['href']))
        if not pageScraped(link.attrs['href']):
            #We have encountered a new page, add it and search it for links
            newPage = link.attrs['href']
            print(newPage)
            getLinks(newPage, recursionLevel+1)
        else:
            print("Skipping: "+str(link.attrs['href'])+" found on "+pageUrl)
getLinks("/wiki/Kevin_Bacon", 0)

cur.close()
conn.close()
```