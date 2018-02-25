## Tornado异步非阻塞

### 一、参考
这里主要学习Tornado牛逼之处，异步非阻塞，其中有异步类库和Future相关知识，很重要

参考链接：
[参考1](http://www.cnblogs.com/wupeiqi/articles/6536518.html)
[参考2](http://www.cnblogs.com/wupeiqi/articles/5702910.html)



### 二、Tornado异步探究之Future

Tornado可以工作在阻塞模式，也可以工作在非阻塞模式

Tornado阻塞模式示例

```
# windows下无法运行，无法调用fork()
import tornado.web
import tornado.ioloop
from tornado.web import RequestHandler
from tornado.httpserver import  HTTPServer

class IndexHandler(RequestHandler):
    def get(self):
        print("start..")
        import time
        time.sleep(5)
        self.write("index page...")
        print("end...")

application = tornado.web.Application([
    (r"/index.html", IndexHandler),
])

if __name__ == "__main__":
    # application.listen(8888)
    # tornado.ioloop.IOLoop.instance().start()

    server = HTTPServer(application)
    server.bind(8888)
    server.start(3) # Forks multiple sub-processes
    tornado.ioloop.IOLoop.current().start()
```


Tornado非阻塞模式示例
```
import tornado.web
import tornado.ioloop
from tornado.web import RequestHandler
from tornado.httpserver import  HTTPServer
from tornado import gen
from tornado.concurrent import  Future
import time

class IndexHandler(RequestHandler):
    @gen.coroutine
    def get(self):
        print("start..")
        future = Future()
        tornado.ioloop.IOLoop.current().add_timeout(time.time() + 5, self.done)  # 模拟耗时IO，没什么卵用
        yield future
        print('end')

    def done(self, *args, **kwargs):
        self.write('async')
        self.finish()

application = tornado.web.Application([
    (r"/index.html", IndexHandler),
])

if __name__ == "__main__":
    application.listen(8888)
    tornado.ioloop.IOLoop.instance().start()
```

来一个有用的示例：

```
import tornado.web
import tornado.ioloop
from tornado.web import RequestHandler
from tornado import gen
from tornado import httpclient

class IndexHandler(RequestHandler):
    @gen.coroutine
    def get(self):
        print("start..")
        http = httpclient.AsyncHTTPClient()
        yield http.fetch('http://www.github.com',self.done)  # http.fetch()返回Future对象，

    def done(self,response, *args, **kwargs):
        print(response)        # response是http://www.github.com返回的结果，用于self.done(response)回调函数
        self.write('返回请求')
        self.finish()

application = tornado.web.Application([
    (r"/index.html", IndexHandler),
])

if __name__ == "__main__":
    application.listen(8888)
    tornado.ioloop.IOLoop.instance().start()
```

有了Future对象就能异步非阻塞了，那么我们就有必要深究下Future到底是什么了？

访问`http://127.0.0.1:8888/index.html`返时Future对象，会被一直hang住，直到`http://127.0.0.1:8888/set`时fu.set_result('666')才返回结果，去执行回调函数，结束请求
```
import tornado.web
import tornado.ioloop
from tornado.web import RequestHandler
from tornado import gen
from tornado.concurrent import  Future

fu = None

class IndexHandler(RequestHandler):
    @gen.coroutine
    def get(self):
        print("疯狂的追求")
        global fu
        fu = Future()
        fu.add_done_callback(self.done)
        yield fu

    def done(self,response, *args, **kwargs):
        print(response)   # set_result时的'666'
        self.write('终于等到你')
        self.finish()

class SetHandler(RequestHandler):
    def get(self):
        fu.set_result('666')
        self.write("只能帮你到这儿了")

application = tornado.web.Application([
    (r"/index.html", IndexHandler),
    (r"/set", SetHandler),
])

if __name__ == "__main__":
    application.listen(8888)
    tornado.ioloop.IOLoop.instance().start()
```

挺好玩的，那么我们用线程再来玩一把,验证Future特性
```
import tornado.web
import tornado.ioloop
from tornado.web import RequestHandler
from tornado import gen
from tornado.concurrent import  Future
from threading import Thread
import time

def task(future):
    time.sleep(5)
    future.set_result('set ok')

class IndexHandler(RequestHandler):
    @gen.coroutine
    def get(self):
        print("疯狂的追求")
        fu = Future()
        fu.add_done_callback(self.done)
        t=Thread(target=task,args=(fu,))
        t.start()
        yield fu

    def done(self,response, *args, **kwargs):
        print(response.result())   # set_result时的'666'
        self.write('终于等到你...')
        self.finish()

application = tornado.web.Application([
    (r"/index.html", IndexHandler),
])

if __name__ == "__main__":
    application.listen(8888)
    tornado.ioloop.IOLoop.instance().start()
```

### 三、Tornado-MySQL实现用户登录示例

参考链接：

http://www.cnblogs.com/wupeiqi/articles/5702910.html

安装模块
是对pymysql的封装，基于Future实现了连接池异步的模块
```
pip3 install Tornado-MySQL
```

app.py代码
```
import tornado.web
import tornado_mysql
from tornado import gen

@gen.coroutine
def get_user(user):
    conn = yield tornado_mysql.connect(host='127.0.0.1', port=3306, user='root', passwd='123', db='db',charset='utf8')
    cur = conn.cursor()
    yield cur.execute("select sleep(10)")
    data = cur.fetchone()
    cur.close()
    conn.close()
    raise gen.Return(data)

class LoginHandler(tornado.web.RequestHandler):
    def get(self, *args, **kwargs):
        self.render('login.html')

    @gen.coroutine
    def post(self, *args, **kwargs):
        user = self.get_argument('user')
        data = yield  gen.Task(get_user, user)
        if data:
            print(data)
            self.redirect('www.digmyth.com')
        else:
            self.render('login.html')

application = tornado.web.Application([
    (r'/login.html', LoginHandler),
])

if __name__ == '__main__':
    application.listen(8008)
    tornado.ioloop.IOLoop.instance().start()
```


login.html 代码
```
    <form method="post">
        <p><input type="text" name="user"></p>
        <input type="submit" value="提交">
    </form>
```

