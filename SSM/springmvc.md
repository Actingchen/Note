# springmvc

## 工作原理

客户端发送HTTP请求，DispatcherServlet控制器拦截到请求，调用处理映射器HandlerMapping 解析请求对应的Handler，处理器适配器HandlerAdapter根据Handler来调用真正Controller处理请求，并处理相应的业务逻辑，Controller返回一个模型视图ModelAndView，ViewResolver进行解析，返回一个视图对象，DispatcherServlet渲染数据对象Model，将得到视图对象返回给客户端，最终显示在相应的页面上。

## 执行流程

![](https://upload-images.jianshu.io/upload_images/5220087-3c0f59d3c39a12dd.png?imageMogr2/auto-orient/strip|imageView2/2/w/1002/format/webp)