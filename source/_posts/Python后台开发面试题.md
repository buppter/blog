---
title: Python后端开发面试题  
date: 2019-03-27 19:08:17  
categories: 面试  
tags:  
    - Python  
    - HTTP     
    - MySQL  
    - 面试  
    
---
整理了自己在Python后台开发求职面试过程中的一些面试题，基本涵盖了Python的基础知识，数据库知识，HTTP协议相关，以及基础的数据结构与算法。也方便自己后续面试的时候回来复习。  
<!--more-->
1. ##### Python的单例   

    ```python
    """使用__new__来实现单例"""
    class Singleton(object):
        def __new__(cls, *args, **kwargs):
            if not hasattr(cls, '_instance'):
                cls._instance = super(Singleton,cls).__new__(cls, *args, **kwargs)
            return cls._instance
    ```

    ```python
    """使用装饰器"""
    def Singleton(func):
        instance = {}
        def wrapper(*args, **kwargs):
            if func not in instance:
                instance[func] = func(*args, **kwargs)
            return instance[func]
        return wrapper
          
    @Singleton
    class Myclass:
        pass
    ```

    ```python
    """使用模块"""
    # singleton.py
    class Singleton():
        def foo(self):
            pass
    
    singleton = Singleton()
    
    #将上边的代码保存为一个模块，然后可以在别的模块中引用
    from singleton import singleton
    
    singleton.foo()
    ```


1. ##### Python的垃圾回收机制  
    Python中的垃圾回收机制主要以`引用计数`为主，`标记-清除`和`分代回收`为辅。  
    `引用计数`的缺点：1. 维护引用计数消耗资源。2. 循环引用问题  
    `标记-清除`：分为两个阶段，第一阶段是标记阶段，把所有的活动对象打上标记；第二阶段把没有被标记的对象进行回收。      从根对象（root object）出发，沿着有向边遍历对象，可达的（reachable）对象标记为活动对象，不可达的对象就是要被清除的非活动对象。在下图中，我们把小黑圈视为全局变量，也就是把它作为root object，从小黑圈出发，对象1可直达，那么它将被标记，对象2、3可间接到达也会被标记， 而4和5不可达，那么1、2、3就是活动对象，4和5是非活动对象会被GC回收。  
    缺点就是清除非活动对象前必须扫描整个堆内存。  

    ![标记清除原理](https://ws1.sinaimg.cn/large/0072atzFly1g1eviz36vxj309q078weh.jpg)   

    `分代回收`：Python将内存根据生存时间划分为不同的集合，每个集合称为一个代。一共有三代，分别为年轻代（第0代），中年代（第1代），老年代（第2代）， 新创建的对象都被分在第0代，当第0代链表数达到上限，Python的垃圾回收机制就会被触发，把那些可以被回收的对象回收掉，不会被回收的就被移到第1代，以此类推。第2代中的对象就是存活时间最长的对象。  

3. ##### Python的生成器、迭代器 
    生成器节约内存，需要的时候才产生结果，而不是立即产生。
    Python中有两种创建生成器的方式：第一种将列表推导式的方括号[]改成圆括号()；第二种是生成器函数，带有yield关键字的函数，使用yield返回值而不是return。  

    ```python
    """生成器表达式"""
    ge = (i**2 for i in range(4))
    print(ge)
    
    for i in ge:
        print(i)
       
    """结果"""
    <generator object <genexpr> at 0x02D5FE70>
    0
    1
    4
    9
    ```

    ```python
    """
    生成器函数
    使用生成器函数实现斐波那切数列
    """
    def fib(n):
        a, b, count = 0, 1, 0
        while count < n:
            yield b
            a, b = b, a + b
            count += 1
        return "Done"
    
    print(fib(5))   
    for i in fib(5):
        print(i)
              
    """输出结果"""
    <generator object fib at 0x0376FE70>
    1
    1
    2
    3
    5
    ```


    生成器有\__next\__()和send()两个方法。f.\__next\__()和next(f)作用都是一样的，打印生成器的下一个结果。send()主要用来向生成器中导入参数。在调用send()方法前需要至少调用一次next()或者有\__next\__()方法。也可以使用send(None)来实现对生成器函数的预激  

   ```python
    def test():
        value = 0
        while True:
            temp = yield value
            if temp == 'e':
                break
            value = "got %s" % temp
        
    t = test()
    print(t.__next__()) # 或者print(next(t))  再或者 print(t.send(None))
    t.send("hhh")
    t.send("aaa")
    t.send("e")   
    
    """运行结果"""
    0
    get hhh
    get aaa
    Traceback (most recent call last):
      File "test.py", line 14, in <module>
        print(t.send("e"))
    StopIteration

    ```


    **注意**：需要注意的一点是在使用`yield from`的时候会自动预激
4. ##### Python的多线程、多进程、协程
16. ##### Python的作用域
3. ##### yield 和 yield from 
    [具体讲解](https://www.cnblogs.com/wongbingming/p/9085268.html)
1. ##### Python中的锁    
    [具体讲解](https://www.cnblogs.com/wongbingming/p/9035575.html)
2. ##### Python的闭包，装饰器 
3. ##### \__init\__ 和 \__new__
4. ##### \*args`和 \**kwages
1. ##### 数据库中的锁  
    乐观锁和悲观锁
2. ##### 数据库中的join  

    ![](https://ws1.sinaimg.cn/mw690/0072atzFly1g1h36cps1kj30qu0l4wi8.jpg)  

    [具体讲解](https://blog.csdn.net/liitdar/article/details/80817087)
3. ##### MySQL的存储引擎  
    InnoDB和MyISAM
4. ##### 数据库的事务  
    [具体讲解](https://www.cnblogs.com/xrq730/p/5087378.html)
5. ##### 数据库的索引 
    分为单行索引和组合索引。其中单行索引又分为普通索引、主键索引（不允许为空，唯一）和 唯一索引（可以为空，但唯一）  
    [具体讲解](https://www.cnblogs.com/wuchanming/p/6886020.html)
6. ##### Redis的数据持久化（AOF和RDB)
7. ##### HTTP 1.0、HTTP 1.1和HTTP 2.0的区别
    [具体讲解](https://www.cnblogs.com/heluan/p/8620312.html)
8. ##### WebSocket  
    WebSocket 是 HTML5 开始提供的一种在单个 TCP 连接上进行全双工通讯的协议。  
    WebSocket 使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。在 WebSocket API 中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输。
9.  ##### HTTP的请求报文和响应报文  
    
    ![请求报文](https://ws1.sinaimg.cn/large/0072atzFly1g1f0tit56rj30cy04lt8p.jpg)

    ![响应报文](https://ws1.sinaimg.cn/mw690/0072atzFly1g1f0wiurchj30gg06fjrs.jpg)
10. ##### 一次完整的HTTP请求过程  
    1. 域名解析  
        1. 首先搜索浏览器自身的DNS缓存
        2. 如果浏览器自身缓存没有，就搜索系统自身的缓存  
        3. 如果系统自身的缓存没有，就搜索host文件
        4. 如果host文件没有，则向本地配置的DNS服务器发起域名解析请求（本地域名服务器）
        5. 如果本地域名服务器依旧没有找到，就会有递归和迭代两种方法解析  
            1. 迭代：  
                a. 本地域名服务器想根域名服务器发起请求  
                b. 根域名服务器返回给本地域名服务器我们该向哪个顶级域名去请求  
                c. 顶级域名服务器想权限域名服务器发请求  
                d. 返回本地域名服务器IP地址  
                e5. 返回给系统内核，最后返回给浏览器    
            2. 递归：  
                a. 本地域名服务器向根域名服务器发请求  
                b. 之后根域名服务器向顶级域名服务器去找  
                c. 顶级域名服务器返回给根域名服务器  
                d. 根域名服务器返回给本地服务器  
                e. 返回给系统，最后返回给浏览器  
    2. TCP连接
    3. 发起HTTP请求
    4. 服务器响应HTTP请求，同时浏览器得到HTML代码
    5. 浏览器解析HTML代码，并请求HTML代码中的资源
    6. 浏览器将页面渲染给用户
11. ##### GET和POST的区别  
    GET是将请求的数据和header一并发送给服务器，只发送一次TCP包  
    POST是先发送header给服务器，然后服务器返回100，再将数据发送给服务器，发送了两次TCP包 
12. ##### TCP三次握手和四次挥手  
13. ##### TIME_WAIT的原因  
    发生在主动请求关闭的一方
14. ##### TCP和UDP的区别 
    1. TCP面向连接，UDP无连接 
    2. TCP提供可靠服务，UDP不保证可靠交付
    3. UDP有较好的实时性
    4. TCP只支持点到点通信，UDP支持一对一，一对多，多对一和多对多通信
    5. TCP对系统资源要求较多，UDP对系统资源要求较少
    6. TCP数据容易发生粘包
15. ##### TCP的粘包  
    1. 发送方引起的粘包是由TCP协议本身造成的，TCP为提高传输效率，发送方往往要收集到足够多的数据后才发送一包数据。若连续几次发送的数据都很少，通常TCP会根据优化算法把这些数据合成一包后一次发送出去，这样接收方就收到了粘包数据。
    2. 接收方引起的粘包是由于接收方用户进程不及时接收数据，从而导致粘包现象。这是因为接收方先把收到的数据放在系统接收缓冲区，用户进程从该缓冲区取数据，若下一包数据到达时前一包数据尚未被用户进程取走，则下一包数据放到系统接收缓冲区时就接到前一包数据之后，而用户进程根据预先设定的缓冲区大小从系统接收缓冲区取数据，这样就一次取到了多包数据。  
    3. 解决办法就是封包、拆包。给一段数据加上包头,这样一来数据包就分为包头和包体两部分内容了(以后讲过滤非法包时封包会加入"包尾"内容)。包头其实上是个大小固定的结构体，其中有个结构体成员变量表示包体的长度，这是个很重要的变量，其他的结构体成员可根据需要自己定义。根据包头长度固定以及包头中含有包体长度的变量就能正确的拆分出一个完整的数据包。
16. ##### select、poll和epoll的区别
    select，poll，epoll本质上都是同步I/O，因为他们都需要在读写事件就绪后自己负责进行读写，也就是说这个读写过程是阻塞的。
    
    select 函数监视的文件描述符分3类，分别是writefds、readfds、和exceptfds。调用后select函数会阻塞，直到有描述符就绪（有数据 可读、可写、或者有except），或者超时（timeout指定等待时间，如果立即返回设为null即可），函数返回。当select函数返回后，可以通过遍历fdset，来找到就绪的描述符。  
    缺点：  
        1. select最大的缺陷就是单个进程所打开的FD是有一定限制的，它由FD_SETSIZE设置，默认值是1024  
        2. 对socket进行扫描时是线性扫描，即采用轮询的方法，效率较低。  
        3. 需要维护一个用来存放大量fd的数据结构，这样会使得用户空间和内核空间在传递该结构时复制开销大。  
    poll本质上和select没有区别，它将用户传入的数组拷贝到内核空间，然后查询每个fd对应的设备状态，如果设备就绪则在设备等待队列中加入一项并继续遍历，如果遍历完所有fd后没有发现就绪设备，则挂起当前进程，直到设备就绪或者主动超时，被唤醒后它又要再次遍历fd。这个过程经历了多次无谓的遍历。 poll没有最大连接数的限制，因为它是采用链表来存储的  
    缺点：  
        1. 大量的fd的数组被整体复制于用户态和内核地址空间之间，对于系统的开销比较大  
        2. poll还有一个特点是“水平触发”，如果报告了fd后，没有被处理，那么下次poll时会再次报告该fd。
 
    **注意**：从上面看，select和poll都需要在返回后，通过遍历文件描述符来获取已经就绪的socket。事实上，同时连接的大量客户端在一时刻可能只有很少的处于就绪状态，因此随着监视的描述符数量的增长，其效率也会线性下降。  
    
    相对于select和poll来说，epoll更加灵活，没有描述符限制。epoll使用一个文件描述符管理多个描述符，将用户关系的文件描述符的事件存放到内核的一个事件表中，这样在用户空间和内核空间的copy只需一次。epoll支持水平触发和边缘触发，最大的特点在于边缘触发，它只告诉进程哪些fd刚刚变为就绪态，并且只会通知一次。还有一个特点是，epoll使用“事件”的就绪通知方式，通过epoll_ctl注册fd，一旦该fd就绪，内核就会采用类似callback的回调机制来激活该fd，epoll_wait便可以收到通知。  
    优点：   
        1. 没有最大并发连接的限制  
        2. 效率提升，不是轮询的方式，不会随着FD数目的增加效率下降  
        3. 内存拷贝，利用mmap()文件映射内存加速与内核空间的消息传递；即epoll使用mmap减少复制开销。  
    在select/poll中，进程只有在调用一定的方法后，内核才对所有监视的文件描述符进行扫描，而epoll事先通过epoll_ctl()来注册一个文件描述符， 一旦基于某个文件描述符就绪时，内核会采用类似callback的回调机制，迅速激活这个文件描述符，当进程调用epoll_wait()时便得到通知。 (此处去掉了遍历文件描述符，而是通过监听回调的的机制。这正是epoll的魅力所在） 

    ![支持一个进程所能打开的最大连接数](https://ws1.sinaimg.cn/mw690/0072atzFly1g1f250wpy4j30wi0gin0s.jpg)
    
    ![FD剧增后带来的IO效率问题](https://ws1.sinaimg.cn/mw690/0072atzFly1g1f2t952tbj30wg0f2wid.jpg)  
        
    ![消息传递方式](https://ws1.sinaimg.cn/mw690/0072atzFly1g1f25ziw99j30wg09emy0.jpg) 
    
    综上，在选择select，poll，epoll时要根据具体的使用场合以及这三种方式的自身特点：  
    1、表面上看epoll的性能最好，但是在连接数少并且连接都十分活跃的情况下，select和poll的性能可能比epoll好，毕竟epoll的通知机制需要很多函数回调。  
    2、select低效是因为每次它都需要轮询。但低效也是相对的，视情况而定，也可通过良好的设计改善。  
17. ##### 几种排序
    [具体讲解](https://www.cnblogs.com/fwl8888/p/9315730.html)
18. ##### 链表
19. ##### 栈，队列
20. ##### 完全二叉树，平衡二叉树，红黑树，B树，B+树，二叉搜索树   
    [平衡二叉树](https://blog.csdn.net/jacke121/article/details/78268602)

    [红黑树](https://www.cnblogs.com/yyxt/p/4983967.html)
    
    [B树，B+树](https://blog.csdn.net/z_ryan/article/details/79685072)  
21. ##### 进程间的通信方式  
    消息队列，信号量，管道，共享内存，Socket  
    
    [具体讲解](https://www.cnblogs.com/zgq0/p/8780893.html)
22. ##### 进程程的状态
    就绪，运行，阻塞  

    [具体讲解](https://www.cnblogs.com/zxf98/p/5716296.html)
23. ##### Docker
    [具体讲解](https://blog.csdn.net/wh211212/article/details/53208960)  
    
24. ##### 网络七层协议，五层协议，TCP/IP四层协议
    七层：物理层，数据链路层，网络层，传输层，会话层，表示层，应用层  
    五层：物理层，数据链路层，网络层，传输层，应用层  
    四层：网络接口层，网络层，传输层，应用层 
  
    [具体讲解](https://blog.csdn.net/cc1949/article/details/79063439)
25. ##### RESRful
    1. 使用HTTPS  
    2. API域名和版本号  
    3. HTTP动词，GET、DELETE、PUT、POST
    4. 过滤，排序，搜索，分页
    5. 状态码和文字说明
    6. 文档  
    
    [具体讲解](https://blog.csdn.net/u013007900/article/details/79875287)
26. ##### MVVC
    多版本并发控制  

    [具体讲解](https://www.cnblogs.com/hirampeng/p/9944200.html)
27. ##### Ping用的什么协议
    ICMP 网络控制消息协议
28. ##### 堆排序
    堆(heap): 父节点的值大于子节点的值的完全二叉树  

    ![求第i个节点的父子节点](https://ws1.sinaimg.cn/mw690/0072atzFly1g1gho0p52kj31rw0tk7gi.jpg)

    ```python
    
    def heapify(alist, n, i):
        if i >= n:
            return
        c1 = 2 * i + 1
        c2 = 2 * i + 2
        max_index = i
        if c1 < n and alist[c1] > alist[max_index]:
            max_index = c1
        if c2 < n and alist[c2] > alist[max_index]:
            max_index = c2
        if max_index != i:
            alist[i], alist[max_index] = alist[max_index], alist[i]
            heapify(alist, n, max_index)
    
    def build_heap(alist, n):
        last_node = n - 1
        last_node_parent = n//2 -1
        for i in range(last_node_parent, -1, -1):
            heapify(alist, n, i)
    
    def heap_sort(alist, n):
        build_heap(alist, n)
        for i in range(n-1, -1, -1):
            alist[i], alist[0] = alist[0], alist[i]
            heapify(alist, i, 0)
    ```

