---
date: 2014/2/23 10:50:08
layout: post
title: 由一个刷票需求实践网络请求处理与多线程编程
thread: 201402231050
categories: 学习
tags: C# Python Network Threading
---

##引子
前几天导师联系到我，有个实在亲戚在某自媒体网站上写了两篇文章参加活动，由于评奖涉及投票，所以就这样产生了一个不太地道的需求：刷票。鉴于参赛选手好多都在刷票，又有导师开口，这个活儿就这么应下了，大家先看看基本需求。

> 基本需求：2月24日0:00之前，尽可能多地给两篇文章点赞，保持文章在排行榜榜首
> 站点的投票规则：不要求登陆或者填写验证码及额外信息，同一个IP在一分钟内只能投一票

好！下面！才开始讲述我和小伙伴这么多天的迭代历程，以前楼主只是看了一些Python的官方文档和简明教程，并没有编写过比较完整的Python程序；C#方面也只是略知皮毛，当然，经过这次刷票这两个东西仍然是略知皮毛，只不过某些类和方法更熟悉了一些= =

##最深刻的体会
** 自身的或用户的需求是推动自学的强大动力 **
** 产品总会有改进的空间，要争取向极致无限逼近 **

##版本一
了解基本规则之后，首先要获取的就是点赞链接，可以借助Chrome的Developer Tools方便的获取，这个工具十分强大，前端工程师的好帮手，常见用法近期会单独整理一篇blog。
最后我们拿到类似的点赞链接：`http://3g.xxx.com/api/saveVotes.go?newsId=15877904&topicId=1802&vote=3639_15461`。楼主比较懒，而且以前也没写过类似的刷票程序，所以第一个想法就是去直接搜索刷票工具，很快可以体验各种XXXVoteTools了。
一般的程序都支持自动更换IP功能，即如果使用了ADSL等拨号连接，程序可以通过不断的拨号、断开来达到快速更换IP的目的，以便在一分钟内能使用更多的IP地址进行投票。这类软件效率一般在20~30票/分钟，而且计算机要直接和入户宽带线链接，不能通过路由，限制颇多。
做得更好些的就是支持使用代理服务器进行投票，文件夹内会有一个文本文件记录很多代理服务器的IP地址，程序循环使用每一条IP进行投票，即我们将请求发送给代理服务器，代理服务器请求投票服务器，然后将请求结果反馈给我们。这种方法只要有源源不断的可用IP地址，按道理效率还是不错的，不过坑爹的事情来了：找了那么久，只找到一款支持代理IP投票的程序，而且这个程序弱到爆，投票请求居然是主线程发出去的！
每一次投票都得等代理服务器响应啊有木有！有的IP不可用，要等好长时间然后报一个超时异常才能换到下一个IP啊有木有！一分钟也就30票，根本不能叫刷票啊！

##版本二
现有程序已经无法满足楼主需求，所以还是自己动手解决问题吧，其实基于这种限制IP投票频率的规则，基本上可以确定是要通大片的过代理服务器进行刷票了。而现有工具的最大问题，其实就在于没有应用多线程了，只需自己实现一个多线程、通过代理服务器访问URL的程序就好了。
刷票程序可能会给客户自己跑，所以编译之后如果是一个exe相对比较方便，而C#工具类丰富，上手快，所以版本二的程序使用C#编写，涉及的包主要有`System.Net`、`System.Threading`。

###核心投票逻辑

    HttpWebRequest request = WebRequest.Create(voteUrl) as HttpWebRequest;
    request.Method = "GET";
    request.UserAgent = DefaultUserAgent;
    request.Timeout = timeout;
    request.Proxy = webProxy;

    try
    {
        HttpWebResponse response = request.GetResponse() as HttpWebResponse;
        TextReader reader = new StreamReader(response.GetResponseStream());
        string responseLine;
        while ((responseLine = reader.ReadLine()) != null)
            Console.WriteLine(responseLine);
        reader.Close();
    }
    catch (Exception e)
    {
        Console.WriteLine("Exception:" + e.Message);
    }

网络操作方面重点就是前面几行，使用HttpWebRequest类，指定好其中的Proxy，Proxy在实例化的时候会指定代理服务器的IP和端口，这类信息从文件中读取就好了。

###实现多线程

    ThreadStart threadStart = new ThreadStart(vote);
    Thread thread = new Thread(threadStart);
    thread.Start();

C#里面多线程的基本实现非常简单，也不用像Java那样继承Thread类或者实现Runnable接口，只需要上面代码那样new一个ThreadStart，传入的是某个方法的名字，然后通过Thread实例开启即可。投票请求的发送和等待响应让子线程去处理，保证了主程序可以很快的更换代理IP，提升刷票效率。

###读取代理服务器地址列表的相关代码

    List<WebProxy> proxyList = new List<WebProxy>();
    StreamReader proxyReader = new StreamReader("proxyList.txt");
    for (string line = proxyReader.ReadLine(); line != null; line = proxyReader.ReadLine())
    {
        string[] ip = line.Split(':');
        proxyList.Add(new WebProxy(ip[0], Int32.Parse(ip[1])));
    }
    proxyReader.Close();

###跑起来看看
每秒十几票的样子，但是如果想要更快呢？那就得多拷贝几份程序，然后每份程序使用不同的代理IP列表，这样多个程序同时运行效率会提高不少。毕竟不太方便，尤其是想要给没有技术基础的客户用。

##版本三
```
					MainThread
VoteThread1		VoteThread2		VoteThread3		…
->proxyList1		->proxyList2		->proxyList3		…
每次投票均使用子线程
```
如果想简化操作，可以考虑按照上面的结构，将程序分为3层，主程序读入代理服务器IP地址，然后平均分成n份，开启n个VoteThread投票线程，传入不同的可用IP列表；VoteThread循环使用代理IP，再开启子线程调用vote方法进行投票，如此一来我们只需要开一个程序就可以达到版本二的多程序效果了。为了方便调整，可以将连接超时时间、开启投票线程个数、每个线程的投票操作间隔放在配置文件里。
直接新建一个App.config文件，内容是这样的：
```
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <appSettings>
    <add key="timeout" value="5000"/>
    <add key="sleep" value="700"/>
    <add key="threads" value="3"/>
  </appSettings>
</configuration>
```
如果想最方便的话，文件名不要改，此时可以在代码中直接使用`ConfigurationManager.AppSettings["xxx"]`来取得相应的配置项，返回值类型为String，记得做相应转换即可。
代理服务器列表的拆分可以使用List类中的`GetRange(startIndex, quantity)`方法，注意数量不能超过`List大小 - startIndex`，否则会抛异常，因此列表如果均分的话要单独处理一下最后一份列表，因为可能比前面几份小。

##版本四
Python！这种刷票需求不正对上Python的胃口么！正好可以把近期对Python的学习实践一下。投票程序的结构与版本三基本相同了，只是使用Python重写一下，感受一下优美的大蟒蛇语言~
###文件操作
Python里面对文件操作真是太方便

    with open("proxyList.txt", "r", encoding="utf-8") as file:
        proxy_list = file.readlines()

就两行！建议使用with这种写法，因为这样写with语句块执行完毕之后Python会关闭文件，做一些清理工作。`readlines()`返回的是一个List，每一项就是文件里面的一行。但是此时每一项结尾会带一个“\n”，这种设计是为了和File类的`writelines(list)`方法统一，因为`writelines()`将列表内容写入文件时，是不给列表每一项结尾加入换行符的。不过这么设计有什么方便之处楼主就不知道了，我们先加上下面这两行来解决问题吧╮(╯▽╰)╭

    for i in range(len(proxy_list)):
        proxy_list[i] = proxy_list[i][0:-1] # 去掉读取文件时带进来的"\n"

###多线程
Python里面多线程模块有两个，一个是比较底层的模块叫`_thread`(Python3与Python2可能叫法不太一样)，另一个是做了封装的`Threading`。这里提出一个新的问题：** 主进程在开启若干个投票线程之后，需要等待子线程运行结束之后才能结束，否则主进程的退出会导致所有子线程停止工作。 **
按照我们C#版本的实现方法，无需其他操作主进程默认会等待子线程的结束。而Python中使用`_thread`模块开启子线程之后，进程是不会等待其运行结束的，要手动通过join等方法来实现。当然，主进程也可以使用while True + time.sleep(60)的方式达到目的。
因此，`_thread`开启的子线程其实比较适用于我们的投票操作，投票操作的主要代码如下：

    def vote(self, proxy, timeout=3):
        """ 投票主逻辑 """
        request_url = "http://3g.xxx.com/api/news/saveVotes.go?newsId=15877904&topicId=1802&vote=3639_15461";
        proxy_handler = request.ProxyHandler({"http": proxy})
        opener = request.build_opener(proxy_handler)
        try:
            print(opener.open(request_url, timeout=timeout).read().decode("utf-8"))
        except Exception as e:
            print(e)

    def run(self):
        while True:
            for proxy in self.proxy_list:
                _thread.start_new_thread(self.vote, (proxy, 10))
                time.sleep(sleep_time)
函数`ran()`中循环代理IP列表，每一票都开启子线程进行投票，投票操作就是执行`vote()`方法，使用`urllib.request`模块，记住流程是build_opener(ProxyHandler)->open(url, timeout)就很容易理解了。
另外注意`_thread.start_new_thread(method, params)`方法，向调用方法method传入的参数类型为tuple。
投票线程使用高级模块`threading`来实现，这样进程默认会等待线程运行结束而不会提前退出。实现方式与Java中有点像，需要定义一个类，继承`threading.Thread`，重写里面的`run()`方法，开启线程的方式变成了这样：

    vote_thread = VoteThread(sub_proxy_list)
    vote_thread.start()

###跑跑看
分分钟万八千票，不给力就加大并发线程、缩短投票间隔、淘到更多IP~~

##产品总有改进的空间
后期的几次改版说实话都是被对手逼出来的，有一个小子一天8w票的追赶我们，到投票截止的前一天更是有刷到几秒钟上千票的速度，敌军真是很狡猾，就等最后爆发，然你没有追赶的可能。好多天建立起来的8w票优势，很快就被追平、反超，我们分析了刷票效率的瓶颈，是我们能够获取到的代理IP数量有限，而且列表更新是靠人工操作，这样效率不高。
所以我们从3家网点购买了代理提取服务，现在这类网站都会提供代理提取API，可以通过代码自动拉去最新的IP列表，我们紧急开发了这样一个工具：
- 新建一个目录，拷贝刷票程序到目录中
- 每30s访问一次代理提取URL，将拿到的IP以追加的方式写到新建目录的proxyList.txt中
- 如果文件中IP数量达到2000，就在新建一个目录，重复前两步的操作

这样一来我们只需要等待代理提取达到一定数量，到相应的文件夹开启新的程序即可。

##刷票是不好的
刷票总归是不好的，不过从中能学习的东西还真不少。
刷票方可以研究如何优化多线程、网络访问等方面的实现；防守方可以研究如何判定刷票行为，如何提高服务器抗压能力。