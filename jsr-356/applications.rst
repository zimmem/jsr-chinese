第二章 应用
=============

Java WebSocket 应用由 websocket endpoint 组成。
websocket endpoint 是表示 websocket 连接两端中的一端的 Java 对象。
正常情况下， 本规范把这样的一个端点视为一个*可编程端点*（programmatic endpoint）。
第二种方法使用 Java WebSocket API 的某些注解来装饰一个 Plain Old Java Object()POJO)对象。
那么实现（容器）将在运行时创建一些合适的对象把这些被注解的POJO发布为 websocket endpoint。
正常情况下， 本规范把这样的端点视为*注解端点*（annotated endpoint）。
规范提到 endpoint 时同时指上面两种类型的 endpoint ， 可编程或注解。
endpoint 在建立 websocket 连接时参与握手过程。
通常， endpoint 将接收和发送各种各样的 websocket 消息。
endpoint 的生命周期在 websocket 连接关闭时结束。

有两个主要的方法可以君子创建 endpoint.  
第一个是实现 Java WebSocket API ， 并按照要求来处理 endpoint 生命周期，
消费和发送消息， 发布等待连接或者连接到另一个终端。

2.1 API 概述
-------------

本节简要介绍了 Java WebSocket API 以便接下来按阶段介绍详细的规定。

2.1.1 Endpoing 生命周期
^^^^^^^^^^^^^^^^^^^^^^^

一个 websocket endpoint 在 Jave WebSocket API 中表示为一个 Endpoint 类实例。
开发者可能通来继承一个 public 类来拦截生命周期中的事件： 连接中、 一个打开的连接关闭、 发生错误。

除非被开发者提供的配置器重写， 否则每个虚拟机中每个应用的链接终端必须使用一个实例来表示。 
[WSC 2.1.1-1] 在经典情况下， 每个 EndPoint 类实例只处理另一个端的连接。

2.1.2 会话（Session）
^^^^^^^^^^^^^^^^^^^^^^

在 Java WebSocket API 模型中用一个 Session 类的实例来表示两个端点间的交互信息序列。 
交互序列从一个打开通知开始， 然后是一些（可能为 0 ） websocket 消息， 
接着是一个关闭通知或可能是一个导致连接中断的致命错误。
每一个与另一个 endpoint 交互的端点， 都有一个唯一的 Session 实例来表示这次交互。
[WSC 2.1.2-1] 这个对应端点连接的 Session 实例被传给 endpoing 实例来代表逻辑 endpoing 生命周期中的关键事件。

开发者可以通过在 Seesion 对象上调用 ``getUserProperties()`` 方法来获取特定会话的具体信息。
Websocket 实现必须保持会话数据至到 endpoint 的 ``onClose()`` 方法结束。[WSC 2.1.2-2]. 
直到这时， websocket 实现才允许销毁开发者数据。 如果 websocket 实现使用会话池来重用 Session 实例，
那么同一个 Session 在代表一个新的连接时需要分配一个新的唯一的会话id。[WSC 2.1.2-3]

Websocket 实现做为已发布容器的一部分， 在机器故障切换时可能需要把 websocket session 从一个结点迁移到另一个结点。
对于被注入到 websocket session 中的开发者数据， 如果这些数据被标记为 ``java.io.Serializable``, 
要求对这些数据进行保持。[WSC 2.1.2-4]


2.1.3 接收消息
^^^^^^^^^^^^^^^^

Java WebSocket API 提供了多种接收消息的方法。 开发者通过实现 MessageHandler 接口， 
把 handler 在 Session 实例上注册， 来注册 Session 实例对应端点上感兴趣的信息。

The API limits the registration of MessageHandlers per Session to be one MessageHandler per native
websocket message type. [WSC 2.1.3-1] In other words, the developer can only register at most one MessageHandler for incoming text messages, one MessageHandler for incoming binary messages, and one
MessageHandler for incoming pong messages. The websocket implementation must generate an error if
this restriction is violated [WSC 2.1.3-2].

API 限制了在一个 Session 上注册 MessageHandler 的数量，
在一个会话上一个消息类型只能注册一个 MessageHandler。[WSC 2.1.3-1]
换句话说， 开发人员只能对文本消息注册一个 MessageHandler， 
对二进制消息注册一个 MessageHandler，
对响应消息（pong）注册一个 MessageHandler。
如果违反这个限制， websocket 实现必须产生一个错误。

在未来的版本中可能会取消这个限制。

2.1.4 发送消息
^^^^^^^^^^^^^^^^

Java WebSocket API 把 Session 会话端点 (endpoing ) 建模为 ``RemoteEndpoint`` 接口的实例。
该接口及其两个子类 (``RemoteEndpoint.Whole``和 ``RemoteEndpoint.Partial``) 包含了多送发送消息的方法。

示例

这里有一个例子： 服务器端等待文本消息， 并且在接收到客户端发送的消息时马上响应。 例子代码如下， 首先只使用 API 类::

	public class HelloServer extends Endpoint {
		@Override
		public void onOpen(Session session, EndpointConfig ec) {
			final RemoteEndpoint.Basic remote = session.getBasicRemote();
			session.addMessageHandler(new MessageHandler.Whole<String>() {
				public void onMessage(String text) {
					try {
						remote.sendText("Got your message (" + text + "). Thanks !");
					} catch (IOException ioe) {
						ioe.printStackTrace();
					}
				} 
			});
		}
	}

下面是使用注解(annotations)的例子::

	@ServerEndpoint("/hello")
	public class MyHelloServer {
		@OnMessage
		public String handleMessage(String message) {
			return "Got your message (" + message + "). Thanks !";
		}
	}

注： 上面两个例子几乎相同， 只是注解的 endpoint 自己进行了路径映射。

2.1.5 关闭连接
^^^^^^^^^^^^^^^

If an open connection to a websocket endpoint is to be closed for any reason, whether as a result of receiving
a websocket close event from the peer, or because the underlying implementation has reason to close the
connection, the websocket implementation must invoke the onClose() method of the websocket endpoint.
[WSC 2.1.5-1]
不管一个的 websocket endpoint 对应的连接以什么原因关闭（收到 websocket 关闭事件消息或底层实现有任何关闭连接的原因），
websocket 都必须调用 endpoint 的 ``onClose()``  方法。

如果关闭事件是由远端端点发起， websocket 实现必须使用以 websocket 协议关闭帧发送过来的关闭代码和原因。
如果关闭事件是由本地容器发起， 比如本地容器认为会话过期， 
本地实现必须使用 websocket 协议关闭代码 1006 （在这代码尤其不能使用在线上），
及一个合适的关闭原因。
这样 endpoing 可以识别到关闭是由远端还是本地发起的。
如果会话是由本地关闭的， 实现必须在调用 ``onClose()`` 方法前发送 websocket 关闭帧。

2.1.6 客户端与服务器
^^^^^^^^^^^^^^^^^^^^

WebSocket 协议是双向协议。 一旦连接建议， 对话的两端是对称的。
一个 WebSocket 客户端和 WebSocket 服务器之间的区别只在于双方通过何种方式连接。
在本规范中， 我们将主动发起连接的一端称为客户端。 
发布（published）并等待连接的一端称为服务器端。
在多数部署情况下， 一个 websocket 客户端只能连接一个 websocket 服务器端， 
一个 websocket 服务器可能接受多个客户端的连接。

因此， WebSocket API 只在配置启动阶段区分客户端和服务器端。

2.1.7 WebSocketContainers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

WebSocket 通过 WebSocketContainer 的实例来表示 websocket 应用。
每个WebSocketContainer实例携带了一些适用于部署其中的端点的配置属性。
WebSocket 实现的服务器端部署中， 每个虚拟机中的每个应用只能有一个唯一的 WebSocketContainer 实例。[WSC 2.1.7-1] 
在客户端部署中， 应用通过 ``ContainerProvider`` 类来获取 ``WebSocketContainer`` 实例。

2.2 使用 WebSocket Annotations 
--------------------------------

Java 注解已被广泛应用于在 Java 对象上添加部署特征， 尤其是在 Java EE 平台上。
Java WebSocket 规范定义了少量 websocket 注解 ， 让开发者可以把 Java 类转换成 websocket endpoint。 
本节提供了简单的概述， 以便接下来在后续介绍详细的要求。

2.2.1 Annotated Endpoints
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**@ServerEndpoint** 注解表示一个 Java 类将在运行时成为 websocket endpoint。
开发者可以使用 ``value`` 属性来指定这个 endpoint 对应的 URI 。 
开发者可以通过  ``encoders`` 和 ``decoders`` 属性来指定把应用对象编码成 websocket 消息及把 websocket 消息解码成应用对象的类。

2.2.2 Websocket 生命周期 
^^^^^^^^^^^^^^^^^^^^^^^^^

在一个已标注 @ServerEndpoint 的类上， 可以使用 @OnOpen 及 @OnClose 来标注在方法上以指定 websocket 实现在接收到一个新连接或当一个连接关闭时， 应该分别调用哪个方法。[WSC 2.2.2-1]

2.2.3 处理消息
^^^^^^^^^^^^^^^

为了使 endpoint 能够处理消息， 开发者需要在方法上标注 @OnMessage 来告诉 websocket 实现，
在收到消息时应该调用哪个方法。[WSC 2.2.3-1]

2.2.4 处理错误 Errors
^^^^^^^^^^^^^^^^^^^^^

为了使被注解的 endpoint 能处理外部事件（ 如解码收到的消息）导致的错误，
可以使用 @OnError 注解来标注在处理错误的方法上。[WSC 2.2.4-1]


2.2.5 Ping 和 Pong
^^^^^^^^^^^^^^^^^^^^

在websocket 协议中使用 ping/pong 机制来检查一个连接是否仍然存活。
按照协议要求， 当 websocket 实现从另一端收到一个 ping 消息，
必须马上响应一个包含同样应用数据的 pong 消息。 [WSC 2.2.5-1]
开发者如果想发送一个单向的 pong 消息， 可以使用 RemoteEndpoint API。
开发者如果想监听 pong 消息， 可以定义一个 MessageHandler，
或者能通过 @OnMessage 注解， 并把消息类型参数设置为 PongMessage。
不管哪种情况， 当容器收到定到该 endpoint 的 Pong 消息， 都必须调用对应的 MessageHandler 或相应标注的方法。
[WSC 2.2.5-2]

2.3 Java WebSocket 客户端 API
--------------------------------

本规范定义了 Java WebSocket API 的两种配置。
Java WebSocket API 用来表示规范中定义的所有功能。
API 可以用于实现独立的 websocket 实现， 或者 Java servlet 容器的一部分， 或 Java EE 平台实现的一部分。
被实现的 API 必须符合 Java  WebSocket API 在 ``javax.websocket.*`` 和``javax.websocket.server.*`` 包中所规定的 api.
当 API 并不是 JAVA EE 平台的部分实现时， 一些Java WebSocket API 的非api的特性是可选的，
比如， websocket endpoint 不是上下文托管 bean 的要求。（见第七章）。
只适用于 Java EE 的特性已被明确标记， 如同所描述。
Java WebSocket API 包含了一个面向桌面， tablet 或智能手机设备的子集。
该子集不包含部署服务器端 ednpoint 的能力。
这个子集就是熟知的Java WebSocket Client API.
这些 API 的实现必须 满足Java WebSocket Client API 在 ``javax.websocket.*`` 包中的 api.


参考条目
------------

[1] I. Fette and A. Melnikov. RFC 6455: The WebSocket Protocol. RFC, IETF, December 2011. See http://www.ietf.org/rfc/rfc6455.txt.

[2] Ian Hickson. The WebSocket API. Note, W3C, December 2012. See http://dev.w3.org/html5/websockets/.

[3] S. Bradner. RFC 2119: Keywords for use in RFCs to Indicate Requirement Levels. RFC, IETF, March
1997. See http://www.ietf.org/rfc/rfc2119.txt.

[4] Danny Coward. Java API for WebSocket. JSR, JCP, 2013. See http://jcp.org/en/jsr/detail?id=356.

[5] Expert group mailing list archive. Web site. See
http://java.net/projects/websocket-spec/lists/jsr356-experts/archive