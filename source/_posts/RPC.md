title: RPC
tags:
  - XML-RPC
categories:
  - RPC
author: kenzhaoyihui
date: 2018-07-07 13:33:00
---
### XML-RPC

RPC (Remote Procedure Call) 远程过程调用， 提供一种“远程接口”来供外部系统调用， 常常用于不同平台，不同架构的系统之间的相互调用， 减少代码的冗余， 便于管理。

XML-RPC 是通过HTTP传输协议，XML作为传送信息的编码格式， 尽可能保持简单， 但同时能够传送，处理， 返回复杂的数据结构。因为是基于HTTP和XML， 所以兼容性比较好， 能够跨域不同的操作系统， 不同的编程语言， 但同时速度就会慢下来。

XML-RPC 一般有分为Server 和Client 端，  Client 向Server发送一个请求体为XML的HTTP POST请求， 被调用的方法在Server端将执行结果以XML格式返回。

Client --> xmlrpclib, 这个模块是用来调用注册在XML-RPC服务端的函数

Server --> SimpleXMLRPCServer， 这个模块用来构造一个最基本的XML-RPC服务器框架


### XML-RPC 简单Python实现

#### Server 端：由于SimpleXMLRPCServer是单线程的，所以当多个客户端同时请求的时候， 就需要用到多线程.


```
#/usr/bin/env python3
 
import os
from SimpleXMLRPCServer import SimpleXMLRPCServer
from SocketServer import ThreadingMixIn
 
def pwd():
    return os.getcwd()
 
def ls(directory=None):
    if directory is None:
        directory = pwd()
    try:
        return os.listdir(directory)
    except OSError as e:
        return e
 
def cd(directory):
    try:
        os.chdir(directory)
    except OSError as e:
        return e
 
    return 'change current working directory to: %s' % (directory)
 
def mkdir(directory):
    try:
        os.mkdir(directory)
    except OSError as e:
        return e
    else:
        return 'successfully create directory: %s' % directory
 
def cp(src, dest):
    with open(src, 'r') as fin:
        with open(dest, 'w') as fout:
            fout.write(fin.read())
    return 'copy %s-> %s' % (src, dest)
 
def hello():
    print "helloworld!"
 
class ThreadXMLRPCServer(ThreadingMixIn, SimpleXMLRPCServer):
    pass
 
class Person(object):
    def __init__(self, name, age):
        self._name = name
        self._age = age
 
    def show(self):
        return str(self)
 
    def __str__(self):
        return 'Person(name=%s, age=%s)' % (self._name, self._age)
#s = SimpleXMLRPCServer(('', 8000), allow_none=True)
 
s = ThreadXMLRPCServer(('', 8000), allow_none=True)
 
s.register_function(pwd) # register function
s.register_function(ls)
s.register_function(cd)
s.register_function(mkdir)
s.register_function(cp)
s.register_function(hello)
 
p = Person('python', 20)
s.register_instance(p)
 
s.serve_forever()
```


#### Client 端:


```
#!/usr/bin/env/python3
 
__author__ = "kenzhaoyihui"
 
from xmlrpclib import ServerProxy
 
s = ServerProxy("http://127.0.0.1:8000")
s.hello()
 
s.pwd()
s.ls()
s.cp('client_test.py', 'client_test_copy1.py')
s.mkdir('tmp1')
s.show()

```

#### Test from Terminal：


```
In [2]: from xmlrpclib import ServerProxy
 
In [3]: x = ServerProxy("http://127.0.0.1:8000")
 
In [4]: x.pwd()
Out[4]: '/home/yzhao_sherry/work/learnpython3/RPC'
 
In [5]: x.ls()
Out[5]: ['server_xml_rpc.py', 'client_test.py']
 
In [6]: x.cp('client_test.py', 'client_test_copy.py')
Out[6]: 'copy client_test.py-> client_test_copy.py'
 
In [7]: x.mkdir('tmp')
Out[7]: 'successfully create directory: tmp'
 
In [8]: x.show()
Out[8]: 'Person(name=python, age=20'
```


#### XML_RPC服务端创建步骤：

1. 导入SimpleXMLRPCServer模块

2. 实例化一个XML-RPC服务对象，在指定的端口监听请求 server =  SimpleXMLRPCServer(("localhost", 8000))

3. 对函数的定义并把该函数注册到server server.register_function(adder_function,'add')##adder_function是服务点定义的函数，add是客户端调用时用的函数。server.register_introspection_functions()##如果用到内部函数，需要把内部函数注册到服务端。 server.register_instance(MyFuncs())##如果要注册的是一个类，可以利用这个函数把类中的方法全部注册到server端。

4. 服务端开始监听运行server.serve_forever()



#### XML_RPC客户端创建步骤：
1. 导入xmlrpclib库模块

2. 创建一个代理到服务端 proxy = xmlrpclib.ServerProxy('http://localhost:8000') 函数参数是URL格式

3. 通过代理就可以调用服务端的方法。