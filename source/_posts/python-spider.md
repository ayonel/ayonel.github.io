---
title: python小爬虫---爬取网名及头像
url: 45.html
id: 45
tags:
  Python
date: 2016-08-04 12:50:33
---

&#160; &#160; &#160; &#160;一款产品发布，其初始用户量很小，这时就需要伪造一批用户来增加平台的用户量，搭起初步的用户生态系统。那么伪造用户时，我们必然需要网名以及头像（也有可能需要签名）等等信息。所以前些日子写了个小爬虫来爬取一批（30000+）网名及头像。 
&#160; &#160; &#160; &#160;对于头像的爬取，我选择的站点是[我要个性网](http://www.woyaogexing.com)，它的图片都没有做防盗链，因此我只需要爬它图片的url就行。对于昵称，我选择的网站是[QQ网名](http://www.oicq88.com)。选择好爬取目标，就需要咱的爬虫登场了。以前爬虫经常用scrapy，但对于我这次的量显然是杀鸡用牛刀。这个爬虫是利用python自带的urllib2，解析网页用的是BeautifulSoup(可用pip安装)，与xpath原理类似。
&#160; &#160; &#160; &#160;下面是头像爬虫源码，代码很简单：

```
# coding:utf-8
import time
import urllib2
import random
from bs4 import BeautifulSoup

def filter(tag):
    if cmp(tag.name, 'img') == 0:
        if tag.has_attr('class'):
            if cmp(tag['class'][0], 'lazy' == 0):
                return True

outfile = open("./20160717/avatar.txt", "a")

for i in range(135, 1500):
    print i
    url = 'http://www.woyaogexing.com/touxiang/index_'+str(i)+'.html'
    response = urllib2.urlopen(url)
    data = response.read()
    soup = BeautifulSoup(data, "lxml")
    imgs = soup.find_all(filter)

    for img in imgs:
        outfile.write(img['src'] + ',' + str(random.randint(0, 1))+ '\n')
    time.sleep(0.5)
```
 
&#160; &#160; &#160; &#160;最后输出的时候在每个头像url后面接了个随机的0或者1，这是用随机数来标志这个人是男生还是女生。
下面是爬取昵称的爬虫：

```
# coding:utf-8
import sys
reload(sys)
sys.setdefaultencoding("utf-8")

import urllib2
from bs4 import BeautifulSoup

def filter(tag):#解析包含网名的标签
    if cmp(tag.name, "ul") == 0:
        if tag.has_attr("class"):
            if cmp(tag['class'][0], 'list') == 0:
                return True

outfile = open("./name.txt", "a")#输出文件

for i in range(55, 145):
    print i
    url = 'http://www.oicq88.com/nvsheng/'+str(i)+'.htm'
    response = urllib2.urlopen(url)#/nvsheng/可以替换为其他的
    data = response.read()
    soup = BeautifulSoup(data, "lxml")
    ul = soup.find_all(filter)
    ulsoup = BeautifulSoup(str(ul[0]), "lxml")
    lis = ulsoup.find_all("li")

    for li in lis:
        outfile.write(str(li.p.text)+'\n')
```

可以直接复制源码进行爬取，前提是`我要个性网`以及`QQ网名`的前端页面没变，为了防止这种情况发生，文末我会附上爬取的源文件链接，大家可以自行下载。 
文件下载链接：[https://ayonel.me/files/name\_avatar\_over.zip](https://ayonel.me/files/name_avatar_over.zip)
