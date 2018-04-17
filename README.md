## What is WebFlux framework
WebFlux framework是Spring5第一代响应式编程框架。
[官方文档](https://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/html/web-reactive.html)

<img src="/docs/image2018-4-14 14_50_43.png" />

架构上spring-webflux与spring-webmvc同级，是spring-webmvc的替代方案，底层网络模型脱离Servlet Api，采用了基于NIO的网络编程框架，支持包括Tomcat，Jetty，Netty等。

spring-webflux依然沿用了与spring-webmvc相同的Controller注解和路由方式，对于旧项目迁移至新项目中带来了便利。

中间层的业务代码由Reactive Stream方式管理，Reactive Streams默认采用Reactor框架（Reactor 入门），同时还支持另一款相对庞大的Reactive Stream框架RxJava。

可以说WebFlux框架是非常灵活的，选择WebFlux作为响应式异步网络编程是明智的选择。

## WebFlux framework搭建

1. $ check out this project
2. $ cd ${product root dir}
3. $ mvn clean install
4. $ mvn spring-boot:run
5. [open browser](http://localhost:8080/product?productId=10)

## Spring MVC差异对比

Spring MVC依然还是沿用Servlet编程模式，Servlet编程模式屏蔽了底层IO模型，所以很多Servlet容器都支持NIO和BIO等多种模式可选，但是Servlet对于线程模型的控制力度很粗。

Servlet网络编程采用的是线程或者线程池模型。在BIO模型下，通过accept模式阻塞，等待客户端请求，当接收到客户端请求后分配一个线程给一个request并采用回调的方式让用户实现servlet.service()方法，即用户在实现service()方法时是在一个分配好的线程中，而不是在accept线程中，accept线程就是所谓的Loop线程。同样在NIO Selector模型下，等待客户端的读写事件，然后为每个事件绑定线程并采用回调的方式让用户实现servlet.service()方法。这种模型的好处是request线程和loop线程进行了有效的隔离，即便是业务代码阻塞也完全不影响loop线程的运行，坏处是线程利用率低下，并发request数越大需要的线程越多，极大的影响了服务的极限性能。举个例子，假设有个业务非常简单，只返回系统当前时间，而系统当前时间通过一个变量维护，通过后台线程不断更新数据，request只简单的返回该变量，这种情况下完全不需要开启request级别线程，在loop线程中就可以直接处理request，只需要1个或几个线程就能应对极大并发的请求。这种业务场景下旧模型没有优势。

WebFlux模式的优势不是在于底层是否采用了NIO还是BIO，而是在上层替换了旧的Servlet线程模型。既然旧模型的问题在于用户无法使用Loop线程，所以WebFlux直接将Controller移交到Loop线程中，所以在Controller层返回的对象必须用Mono<T>或者Flux<T>包裹。这样做的好处在于允许用户在Loop线程中进行一些快速的非阻塞的操作，比如定义响应式编程模型对象等，不阻塞Loop线程，并绑定Scheduler，保证响应式编程模型能在新的线程中执行，提高并发性能。

将项目以Debug方式启动后，可以将断点设置在Controller入口的第一行代码，并用多个浏览器同时请求可以发现和Spring MVC的差异，当WebFlux设置Loop线程=1时，只有第一个request能进入Controller层，其他线程会在第一个request运行完所有Controller代码后才能依次被调用。而Spring MVC可以允许多个request并发调用Controller层代码。