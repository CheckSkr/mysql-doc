
#  Agenda

**1.  Design A WebApp Server**

**2.   Tomcat Architecture**

**3.   Tomcat Connectors ( BIO NIO  APR)**
```
a. why nio?
b. why not netty?
c. Asynchronous servlet
```
**4. Tomcat Servlet containers**

**5. Tomcat Design Pattern**




# 1.  Design A WebApp Server

1. Thinking of： 组成和生命周期

 ![image](https://raw.githubusercontent.com/qhwj2006/mysql-doc/master/res/mytomcat.png)

2. Deeper: 多协议支持 和 request/response
 ![image](https://raw.githubusercontent.com/qhwj2006/mysql-doc/master/res/mytomcat1.png)

3. And Deeper：Servlet Specification
 ![image](https://raw.githubusercontent.com/qhwj2006/mysql-doc/master/res/mytomcat2.png)


4. What's More

>  如何以Daemon进程的 式运 

> 如果优雅的关闭Server进程
 
> 是否提供JMX

> 日志如何处理

> 是否可配置?如何配置

> 是否可扩展?扩展点，扩展方式

> 具体支持哪些协议

> IO模型:同步/异步?阻塞/非阻塞?

> 实现servlet规范 的要求， 量的细节
 


# 2. Tomcat Architecture
2.1.   started with tomcat conf serverxml configuration
 
 ```
<?xml version='1.0' encoding='utf-8'?>
<Server port="8005" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
  <Listener className="org.apache.catalina.core.JasperListener" />
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />
  <GlobalNamingResources>
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>
  <Service name="Catalina">
        <Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000"  redirectPort="8443" />
        <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
        <Engine name="Catalina" defaultHost="localhost">
              <Realm className="org.apache.catalina.realm.LockOutRealm">
                    <Realm className="org.apache.catalina.realm.UserDatabaseRealm"resourceName="UserDatabase"/>
              </Realm>
              <Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="true">
                    <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"  prefix="localhost_access_log." suffix=".txt" pattern="%h %l %u %t &quot;%r&quot; %s %b" />
              </Host>
          </Engine>
  </Service>
</Server>
 ```
###  simplified format:
```
<Server>                                                //顶层类元素，可以包括多个Service   
    <Service>                                           //顶层类元素，可包含一个Engine，多个Connecter
        <Connector>                                     //连接器类元素，代表通信接口
                <Engine>                                //容器类元素，为特定的Service组件处理客户请求，要包含多个Host
                        <Host>                          //容器类元素，为特定的虚拟主机组件处理客户请求，可包含多个Context
                                <Context>               //容器类元素，为特定的Web应用处理所有的客户请求
                                </Context>
                        </Host>
                </Engine>
        </Connector>
    </Service>
</Server>
```

**Architecture View：**
 
![image](https://github.com/qhwj2006/mysql-doc/raw/master/res/tocmat.jpg)


> Catalina: tomcat的顶级容器，main()方法中就是通过，创建Catalina 对象实例，来启动或者关闭 tomcat；

> Server: 是管理tomcat所有组件的容器，包含一个或多个的service；

> Service: Service是包含Connector和Container的集合，Service用适当的Connector接收用户的请求，再发给相应的Container来处理；

> Connector: 主要功能是 ◇socket的接收 ◇根据协议类型处理socket ◇封装相应的request和response，交给Container；
 

> Container: Engine容器接收来自Connector的请求，并且通过Pipeline依次传递给子容器的Pipeline；

> Engine: 在Engine的Pipeline中的Valve的invoke方法中，根据request.getHost()来定位下一个host；

> Host: 一个Web服务器虚拟机，管理着具体的 web application；

> Context: 就是我们所部属的具体Web应用的上下文，每个请求都是在具体的上下文中处理；

> Wrapper：对应着Web的每一个 Servlet；
 
 
 
 **A Http Request&Response：**
 
 ![image](https://raw.githubusercontent.com/qhwj2006/mysql-doc/master/res/tomcatRR.png)
 
 
 
 
 
    
  
 # 3. Tomcat Connectors 

###  1. Blocking / Non-blocking IO
   
```
QUESTION: Blocking / Non-blocking? Blocking Where?

In simple words :
blocking IO is something like "you waiting for your girl to join you on a date, you wait for her indefinitely".
non-blocking IO is like "you have asked someone to join you on a date, but you are not sure if she turns up so early, 
so you decide to do other works pending, or sometimes you get bored and may try asking another girl for a date".
（from：https://www.quora.com/In-simple-terms-what-is-blocking-and-non-blocking-IO）
```






### 2. Tomcat Connectors
```
Connector是网络socket相关接口模块，它包含两个对象，ProtocolHandler及Adapter 
ProtocolHandler是接收socket请求，并将其解析成HTTP请求对象，可以配置成nio模式或者传统io模式
Adapter是处理HTTP请求对象，它就是从StandEngine的valve一直调用到StandWrapper的valve.

逻辑功能：
1. 接收socket
2. 从socket获取数据包，并解析成HttpServletRequest对象
3. 从engine容器开始走调用流程，经过各层valve，最后调用servlet完成业务逻辑
4. 返回response，关闭socket
```

###  3. Tomcat BIO  Connectors(Java Blocking I/O)

![image](https://raw.githubusercontent.com/qhwj2006/mysql-doc/master/res/tomcatBIO.jpg)

上图可知,主要分成3个模块：
 
-  Http11Protocol: 包含了JIoEndpoint对象及Http11ConnectionHandler对象。
 
```
Http11ConnectionHandler :维护了一个Http11Processor对象池，Http11Processor对象会调用CoyoteAdapter完成http request的解析和分派。

JIoEndpoint : 维护了两个线程池，Acceptor及Worker。Acceptor是接收socket，然后从Worker线程池中找出空闲的线程处理socket，
如果worker线程池没有空闲线程，则Acceptor将阻塞。Worker是典型的线程池实现。Worker线程拿到socket后，就从Http11Processor对象池中获取
Http11Processor对象 
```
- Mapper

```
维护了一个从Host到Wrapper的各级容器的快照。它主要是为了，当http request被解析后，能够将http request绑定到相应的servlet进行业务处理。
```

- CoyoteAdapter
```
负责将http request解析成HttpServletRequest对象，之后绑定相应的容器，然后从engine开始逐层调用valve直至该servlet。

code: connector.getContainer().getPipeline().getFirst().invoke(request, response);
```

###  4. Tomcat NIO  Connectors(Java Non-blocking I/O )


![image](https://raw.githubusercontent.com/qhwj2006/mysql-doc/master/res/tomcatNIO.jpg)

从上图可以看出：基本流程和BIO的类似，唯一不同的是：
  JIoEndpoint 和 NioEndPoint；
  NioEndPoint 的处理流程：
  
![image](https://raw.githubusercontent.com/qhwj2006/mysql-doc/master/res/tomcatNIOEndpoint.jpg)


**NIO的模式的实现模型如下：**

> Acceptor线程：全局唯一，负责接受请求，并将请求放入Poller线程的事件队列。Accetpr线程在分发事件的时候，采用的Round Robin的方式来分发的

> Poller线程：官方的建议是每个处理器配一个，但不要超过两个，由于现在几乎都是多核处理器，所以一般来说都是两个。每个Poller线程各自维护一个事件队列（无上限），它的职责是从事件队列里面拿出socket，往自己的selector上注册，然后等待selector选择读写事件，并交给SocketProcessor线程去实际处理请求。

> SocketProcessor线程：Ali-tomcat的默认配置是250(参见server.xml里面的maxThreads)，它是实际的工作线程，用于处理请求。




###  5. Tomcat APR Connectors(Apache Portable Runtime)

  ```
  to be continued
  ```

###  6. Connector Comparison

    types | Java Blocking Connector | Java Non Blocking Connector | APR/native Connector
---|---|---|---
Classname | Http11Protocol | Http11NioProtocol  | Http11AprProtocol 
Tomcat Version  | 3.x onwards  | 6.x onwards   | 5.5.x onwards 
Support Polling  | NO | YES  | YES 
Polling Size  |   N/A   | maxConnections  | maxConnections 
Read Request Headers | Blocking  | Non Blocking  | Blocking 
Read Request Body  | Blocking | sim Blocking  | Blocking 
Write Response  | Blocking | sim Blocking  | Blocking 
Wait for next Request  | Blocking  | Non Blocking  | Non Blocking 
SSL Support | Java SSL | Java SSL   | OpenSSL 
 SSL Handshake  | Blocking | Non blocking  | Blocking 
Max Connections | maxConnections | maxConnections  | maxConnections 



图说明：
 该图 来自于 [tomcat doc](http://tomcat.apache.org/tomcat-7.0-doc/config/http.html#Connector_Comparison);
 
**Caution:**  Wait for next Request 这一项。它表示的是当开启keep-alive的情况下三种模式对等待下一次请求是否阻塞。

-   BIO/NIO/APR 模式下的keep-alive实现 [参见](https://yq.aliyun.com/articles/20307)

 


> 

---
>  caution :  the difference of "TCP and http Keep-alive


```
TCP的keep alive是检查当前TCP连接是否活着；HTTP的Keep-alive是要让一个TCP连接活久点。它们是不同层次的概念。
    TCP keep alive的表现：
    当一个连接“一段时间”没有数据通讯时，一方会发出一个心跳包（Keep Alive包），如果对方有回包则表明当前连接有效，继续监控。
这个“一段时间”可以设置。
```

---


# 4. Tomcat Servlet containers



to be continued(下回分解)




# 5. Tomcat Design Pattern


### 1. 外观模式Facade

```
definition:外观模式封装了子系统的具体实现，提供统一的外观类给外部系统，这样当子系统内部实现发生变化的时候，不会影响到外部系统。
```
tomcat 中用到该模式的地方：

![image](https://raw.githubusercontent.com/qhwj2006/mysql-doc/master/res/tomcatFacade1.png)


### 2. 观察者模式Publish&Subscribe
    
```
definition: 观察者模式是一种非常常用的模式，比如大家熟悉的发布-订阅模式，客户端应用中的事件监听器，以及通知等其实都属于观察者模式。
观察者模式主要是在当系统中发生某个状态变更或者事件的时候，有另外一些组件或者对象对此次变化有兴趣
，这个时候那些对变化感兴趣的对象就可以做为观察者对象来监听变化，而被观察对象要负责发生变化的时候触发通知操作。
```
tomcat 中用到该模式的地方：
![image](https://raw.githubusercontent.com/qhwj2006/mysql-doc/master/res/tomcatPubSub.png)



### 3. 责任链模式Chain of Responsibility

```
definition: 通过名称我们应该就能知道责任链模式是解决啥问题的？当我们系统在处理某个请求的时候，请求需要经过很多个节点进行处理，
每个节点只关注自己的应该做的工作，做完自己的工作以后，将工作转给下一个节点进行处理，直到所有节点都处理完毕。
责任链模式在日常生活中例子挺多，比如快递，当你发一个从深圳到北京的快递的时候，你的包裹会从一个分拨中心传递到
下一个分拨中心，直到目的地，这里面每个分拨中心都是链路上的一个节点，它做完自己的工作，然后将工作传递到下一个节点，
还比如路由器中传递某个数据包其实也是同样的思路。
```
tomcat 中用到该模式的地方：
![image](https://raw.githubusercontent.com/qhwj2006/mysql-doc/master/res/tomcatChain.png)




### 4. 模板方法模式Template Method 

```
definition: 模板方法模式抽象出某个业务操作公共的流程，将流程分为几个步骤，其中有一些步骤是固定不变的，有一些步骤是变化的，固定不变的步骤通过一个基类来实现，而变化的部分通过钩子方法让子类去实现，这样就实现了对系统中流程的统一化规范化管理。在日常生活中其实也有类似的例子，比如我们知道的连锁加盟店，他们都是有固定的加盟流程，只不过每一家店开的时候，店的选址，装修等不同的而已，但是总体的加盟流程已经是确定的。
```
tomcat 中用到该模式的地方：
![image](https://raw.githubusercontent.com/qhwj2006/mysql-doc/master/res/tomcatTemplate1.png)


> Tomcat中关于生命周期管理的地方很好应用了模板方法模式，在一个组件的生命周期中都会涉及到init(初始化)，start（启动），stop(停止)，destory（销毁），而对于每一个生命周期阶段其实都有固定一些事情要做，比如判断前置状态，设置后置状态，以及通知状态变更事件的监听者等，而这些工作其实是可以固化的，所以Tomcat中就将每个生命周期阶段公共的部分固化，然后通过initInternal,startInternal,stopInternal,destoryInternal这几个钩子方法开放给子类去实现具体的逻辑。

