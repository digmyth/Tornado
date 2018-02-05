# Tornado
Tornado是一个web框架,可以工作在阻塞模式或异步非阻塞模式，因为异步非阻塞工作模式Tornado非常出名，速度相当快，同时可以当作异步非阻塞IO模块来用.
值得花时间来学习它

## Tornado 基础知识学
Django:
  socket:     无，  wsgiref
   中间件：    有
 路由系统：    有
视图函数：     有
    ORM：     有
模板引擎：     有
simple_tag:   有
   cookie:    有  
  session:    有
    csrf:     有
    xss:      有
    其它：缓存，信号，Form, ModelForm,Admin
    
 Tornado:
     socket:    有， 有wsgiref    (Tornado就是用了自有socket实现异步非阻塞和websocket,Tornado如果用了wsgiref就没有异步非阻塞和websocket功能)
     中间件：    没有（自己写）
    路由系统:    有
    视图函数：   有
    uimethod:   有（相当于simple_tag）
    uimodule:   有（相当于simple_tag）
     cookie:    有  
    session:    有
       csrf:    有
        xss:    有
        其它：  没有

### 基本应用




### 异步非阻塞



### 原生WebSocket
Tornado原生支持Websocket


## 自定义Session组件（公共组件，Tornado为例）


## 自定义Form验证组件（公共组件，Tornado为例）


