---
title: 基于协程、异步IO的python爬虫
url: 290.html
id: 290
tags:
  Python
date: 2017-05-17 22:32:14
---

&#160; &#160; &#160; &#160;协程，又称微线程，纤程。英文名Coroutine
&#160; &#160; &#160; &#160;我们可以将协程理解为一个子程序/函数的特例。与子程序/函数不同的是，子程序调用总是一个入口，一次返回，调用顺序是明确的。而协程的调用和子程序不同。协程看上去也是子程序，但执行过程中，在子程序内部可中断，然后转而执行别的子程序，在适当的时候再返回来接着执行。 
&#160; &#160; &#160; &#160;子程序与主程序是调用与被调用关系，而协程之间是平等的关系，先执行一个协程A，在适当的时候去执行另一个协程B，在适当的时候又返回协程A继续执行。协程在单线程中实现了类似多线程的效果，但因为协程始终属于一个线程，不同协程的执行只是一个线程在执行不同的“函数”而已，因此它省去了线程切换带来的开销，具有很高的并发和性能。
&#160; &#160; &#160; &#160;但协程是一个线程执行，如何利用多核CPU呢，答案就是协程+进程，每个核维持一个进程，在该进程中有一个线程来执行协程，这样就能充分利用多核以及协程的高效率，获得极大地性能。
&#160; &#160; &#160; &#160;在Python中，由于GIL（Global Interpreter Lock）的存在，导致在python中的多线程其实是伪多线程，同一时刻只有一个线程正在执行。因此在python中多线程往往（对于CPU密集型）还没有单线程高效，（当然可以用multiprocessing模块，利用多进程实现多核CPU并行），因此python的多线程对于CPU密集型工作其实是个鸡肋。多进程+协程才是python性能的大杀器。在IO密集型工作中，python的多线程能发挥得好一些，但还是没有协程高效。
&#160; &#160; &#160; &#160;回到本文实现的目标：做一个爬虫。爬虫属于IO密集型，那么如果我们不利用任何多线程或者协程技术，也就是使用同步来实现爬虫，那么对于每一次网络调用，CPU都会阻塞，直到网络调用response的到来。这就使CPU的大部分时间阻塞在等待网络IO上，效率及其低。而利用协程的话，每当一个协程进行网络调用时，CPU不用阻塞等待而是直接执行下一个协程，当网络调用返回后，将注册在事件循环中，在未来的某个时刻接着执行。这就是异步IO，能极大地提高性能。
&#160; &#160; &#160; &#160;**Scrapy**框架使用多线程来实现爬虫这种IO密集型的工作，也能取得不错的性能。但我们今天不打算利用线程池来实现高效爬虫，而是使用协程来实现。Python 3.5以后已经直接支持async, await关键字，使得协程的实现变得很简单。 
&#160; &#160; &#160; &#160;回想一下如果我们不使用协程+异步IO，我们将这么实现,代码基于python3.5：
``` python
from urllib import request
from urllib.error import HTTPError
import json

def crawl(url):
    try:
        res = request.urlopen(url)
    except HTTPError:
        return None
    return res.read() //返回字节流
if name == 'main':
    url = 'https://api.github.com/users/ayonel/followers?per_page=99&page=1'
    res_data = crawl(url)
    json_data = json.loads(res_data.decode()) // 字节流解码，并转为json
    print(json_data)

```

&#160; &#160; &#160; &#160;上面这段代码，利用python3内置的urllib模块，爬取github API, 返回的是[笔者的github](https://github.com/ayonel)的粉丝（虽然没几个...）。 在上面代码中，程序将阻塞在这一行，直到网络调用返回结果。
`res = request.urlopen(url)`
&#160; &#160; &#160; &#160;我们的协程+异步IO爬虫，就是解决这个问题，当发出网络请求，程序不用阻塞，CPU直接执行下一个协程。
&#160; &#160; &#160; &#160;在urllib中的request是基于同步实现的，我们需要[aiohttp](http://aiohttp.readthedocs.io/en/stable/)这个模块，用来发出异步的网络请求。另外，我还将爬取到的结果，存入了MongoDB，但是python中最常用的操作MongoDB的驱动pymongo也不是基于异步实现的，还好有[motor](http://motor.readthedocs.io/en/stable/),一个基于异步IO的python MongoDB驱动。
&#160; &#160; &#160; &#160;还是原来的需求，我们需要爬取github上一些用户的粉丝。爬取链接为：
`https://api.github.com/users/{user}/followers?per_page=99&page={page}` 
&#160; &#160; &#160; &#160;上述这个URL返回一个用户的粉丝，并且每页最多返回99个，如果超过99个，则需要对**page**+1 先定义两个协程函数， fetch 以及 do_insert，分别用于异步爬取网页，以及异步插入MongoDB。
```
async def fetch(session, user, page):
    url = 'https://api.github.com/users/' + user + '/followers?per_page=99&amp;page=' + str(page)
    with async_timeout.timeout(20):  // 默认超时时间20s
        async with session.get(url, headers={}) as response:
            if response.status == 200:
                return True, user, await response.read()  // True代表爬取成功
            else:
                return False, user, await response.read() // False代表爬取失败
```
还有插入数据库函数：
```
async def do_insert(arg):
    await collection.insert_one(arg)  # collection是全局变量，代表你要插入的MongoDB的collection
```

&#160; &#160; &#160; &#160;上面这两个协程函数，是我们异步IO调用的关键函数，也是我们爬虫性能高效的关键所在。
&#160; &#160; &#160; &#160;有了协程函数，我们就需要定义一个主函数来执行，我们的主函数也是协程函数，另外还传入了你要爬取的用户集
```
async def main(loop, peopleSet):
    async with aiohttp.ClientSession(loop=loop) as session:
        for people in peopleSet:
            page = 1
            while True:  // 因为有些人不止有99个粉丝，所以遇到一页返回99个粉丝的，不能直接下一个人，还需要将page+1，去爬取该人的其他粉丝
                is_ok, user, data = await fetch(session, people, page)
                print(user)
                if is_ok:  // 如果网络返回正常，status==200
                    if data[0] == 31 and data[1] == 139:  // 这儿是个坑，判断是不是gzip压缩过的字节流。原因见下文解释
                        print('gzip data returned!')
                        data = zlib.decompress(data, zlib.MAX_WBITS | 16)  // 解gzip字节流

                follower_list = []
                for follower in json.loads(data.decode()): // 将字节流解码为str，并转为json
                    follower_list.append(follower['login'])

                await do_insert({'user': user, 'follower': follower_list}) // 异步调用插入数据库
                if len(follower_list) == 99: // 如果返回99个粉丝，需要page+1
                    page += 1
                else: // 否则，直接break,爬取下一个人
                    break
            else: // 网络调用失败，直接break，爬取下一个人
                break
```
&#160; &#160; &#160; &#160;解释一下上面为什么要判断字节流是否是gzip压缩过的。Github API 有个坑，它有时会将响应内容压缩为gzip返回，我们用浏览器查看时没问题，因为浏览器会自动判断是否是gzip格式的字节流，如果是将会自动解压。而aiohttp返回的response.read()是原始的字节流，你不知道它有没有经过gzip压缩。按常理来说，如果你在你的请求头中不要设置Accept-Encoding字段，服务器应该认为该客户端不接受任何压缩过的内容，应该返回未压缩的字节流。但我强制指定请求头，不设置Accept-Encoding字段，Github API还是玄幻地随机返回gzip字节流。并且它的response的headers中永远显示其Content-Encoding = gzip，这儿应该是个bug吧。所以我不得不自行判断是否是gzip格式的，判断方法很简单，利用字节流的前两个字节： 
&#160; &#160; &#160; &#160;前两个字节用于标识gzip文件，其中第一个字节ID1 = 31（0x1f，\\037），第二个字节ID2 = 139（0x8b，\\213），如果判断该字节流以这两个字节开头，那么可以初步认为这是gzip字节流。
&#160; &#160; &#160; &#160;我们还需要开启事件循环，才能达到异步IO的效果，完整程序见下文： 
```
'''
Author: ayonel
Date:
Blog: https://ayonel.me
GitHub: https://github.com/ayonel
E-mail: ayonel@qq.com
提取出所有pull中的人员，爬取其关注关系。
'''
import itertools
import json
import aiohttp
import asyncio
import time
from motor.motor_asyncio import AsyncIOMotorClient
import zlib
import async_timeout

async_client = AsyncIOMotorClient('localhost', '27017') // 建立motor的client
db = async_client['follow'] // 指定要插入的数据库
collection = db['asyncfollow'] // 指定要插入的collection

//异步插入函数

async def do_insert(arg):
    await collection.insert_one(arg)

//异步爬取函数

async def fetch(session, user, page):
    url = 'https://api.github.com/users/' + user + '/follower?per_page=99&amp;page=' + str(page)
    with async_timeout.timeout(20):  // 默认超时时间20s
        async with session.get(url, headers={}) as response:
            if response.status == 200:
                return True, user, await response.read()  
            else:
                return False, user, await response.read()  

//主函数

async def main(loop, peopleSet):
    async with aiohttp.ClientSession(loop=loop) as session:
        for people in peopleSet:
            page = 1
            while True:  // 因为有些人不止有99个粉丝，所以遇到一页返回99个粉丝的，不能直接下一个人，还需要将page+1，去爬取该人的其他粉丝
                is_ok, user, data = await fetch(session, people, page)
                print(user)
                if is_ok:  // 如果网络返回正常，status==200
                    if data[0] == 31 and data[1] == 139:  // 这儿是个坑，判断是不是gzip压缩过的字节流。原因见下文解释
                        print('gzip data returned!')
                        data = zlib.decompress(data, zlib.MAX_WBITS | 16)  // 解gzip字节流

                follower_list = []
                for follower in json.loads(data.decode()): // 将字节流解码为str，并转为json
                    follower_list.append(follower['login'])

                await do_insert({'user': user, 'follower': follower_list}) // 异步调用插入数据库
                if len(follower_list) == 99: // 如果返回99个粉丝，需要page+1
                    page += 1
                else: // 否则，直接break,爬取下一个人
                    break
            else: // 网络调用失败，直接break，爬取下一个人
                break


if name == 'main': // 程序入口
    start = time.time() // 标记开始时间
    peopleSet = set() // 初始化需要爬取的用户集
    peopleSet.add('ayonel')
    peopleSet.add('malash')
    // 上面我只添加了2个人，实际我的peopleSet中有2万多人
    loop = asyncio.get_event_loop() // 开启事件循环
    try:
        loop.run_until_complete(main(loop, peopleSet)) // 开始执行main
    except Exception:
        pass
    finally:
        print('总耗时：' + str((time.time()-start)/60))
```
&#160; &#160; &#160; &#160;我另外还是实现了同步的urllib.request作为对比测试，实验显示，在我的异步爬虫爬取了9000+人的时间里，同步爬虫爬取了300人不到。性能差距是巨大滴...
&#160; &#160; &#160; &#160;上面完整代码给出的爬虫还能不能提高速度呢？答案是可以的！上述代码中其实把整个爬取任务封装成了一个大协程，这个大协程由许多个网络调用小协程以及数据库插入小协程构成。同时我们的所有aiohttp的请求复用了同一个aiohttp.ClientSession(),这样做的好处是我们的程序运行期间只有一个ClientSession对象，十分节约内存资源，如果我们对每个请求新开一个ClientSession的话，请求一多，内存直接就爆掉了。我实际了测试20000个请求，很快内存就会吃到10G，所以说对于ClientSession我们需要复用，那是不是所有请求都复用一个呢，就像上面完整代码中一样。当然不是，我们可以自己组织，比如500个请求复用一个ClientSession对象，或者200个请求复用一个ClientSession对象，在实际测试中，爬取效率会迅速提升，远远高于所有请求复用同一个ClientSession对象的情况。
&#160; &#160; &#160; &#160;aiohttp中还有connector（连接池），相信利用connector肯定还能更进一步提高爬取效率。 最后总结，协程+异步IO是提高并发的大杀器，python一直被诟病的性能问题，可以从这儿找回点自信，但是缺点也比较明显，只适合于IO密集型的工作，另外异步代码确实不如同步代码好理解，虽然3.5已经实现了async await关键字，但还是需要多coding才能加深理解。 听说tornado是个不错的东东，有时间可以看看。