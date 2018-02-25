## Tornado异步非阻塞.

这里主要学习Tornado牛逼之处，异步非阻塞，其中有异步类库和Future相关知识，很重要

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
        tornado.ioloop.IOLoop.current().add_timeout(time.time() + 5, self.done)
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
