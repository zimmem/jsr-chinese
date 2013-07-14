第三章 配置
===========

WebSocket 应用通过几个关键参数进行配置： 
在窗口中识别 websocket endpoint 的路径映射；
endpoing 支持的子协议；
应用要求的扩展。
另外， 在进行握手时， 应用可能会执行其它的配置任务，
如检查客户端的主机名， 或者处理 cookie。
本章详述在容器上如何支持这些配置任务。

3.1 服务器端配置
----------------

为了发布一个能被客户端连接的可编程 endpoint, 窗口需要一个 ``ServerEndpointConﬁg`` 实例。
这个对象保持配置数据， 默认实现提供配置 endpoint 所需要的算法。
WebSocket API 允许开发者提供一个实现自 ``ServerEndpointConﬁg`` 的自定义 ``ServerEndpointConﬁg.Conﬁgurator``，
来重写这些配置操作。[WSC-3.1-1]

以下列出这些操作。

3.1.1 URI 映射
^^^^^^^^^^^^^^^^^^

本节描述服务器端的 URI 映射策略。
WebSocket 实现必须把接收到的 URI 跟所有的 endpoint path 比较， 并决定哪一个最符合。
在握手过程，
如果传入的 URI 与某个 endpoint 的 相对路径精确匹配，
或者如果某个结点的路径是一个 URI 模板且与传入的 URI 是该模板的实例，
那么传入的 URI 匹配该 endpoint 。 [WSC-3.1.1-1]

一个应用中不能有多个 endpoint 使用相同的相对路径。 
一个应用中不能有多个 endpoint 使用对待的路径模板。
[WSC-3.1.1-2]

然而， 理论上一个握手请求中传入的 URI 仍有可能同时匹配多个 endpoint， 比如下面的例子

传入 URI : "/a/b"

Endpoint A 映射到 "/a/b"

Endpoint B 映射到  "/a/{customer-name}"

Websocket实现并尝试以如下方式把传入的 URI 匹配到 endpoint 路径上（URI 或 一级 URI 模板 ）：[WSC-3.1.1-3]

不管 endpoint 路径是 URI 或一级URI模板， 只要片段数量不一致， 就不会匹配， 片段由 "/" 分割。
因些， 容器将遍历和片段数相同的 endpoint 路径, 从左往右与 输入 URI 比较相应的片段。
对于每一个片段， 容器将记住是否精确匹配， 如果是一个变量片段， 那么检查下一个。
如果在这个过程结束时， 仍有一个 endpoint 路径， 那么将产生一个配对。

Because of the requirement disallowing multiple endpoint paths and equivalent URI-templates, and the
preference for exact matches at each segment, there can only be at most one path, and it is the best match.

因为规范要求不允许多个 endpoint 路径和等价 URI 模板， 而在匹配过程优先选择精确匹配， 因些最终只能有一个最佳匹配。

比如：

#. 假设 endpoint 的映射路径是 ``/a/b/`` ,那么只有一个输入 URI 能跟它匹配， 那就是 ``/a/b``.
#. 假设 endpoint 映射到 ``/a/{var}``
   	能被匹配的 URI 为 ``/a/b`` (var=b), ``/a/apple`` (var=apple)

   	不能被匹配的 URI 为 ``/a`` , ``/a/b/c``
   	因为空字符串及保留字符 ``/`` 不是有效的一级 URI 模板展开（这翻译有点别扭））
#. 假设我们有三个 endpoint ， 他们映射的路径分别为：
	
	* endpoint A : ``/a/{var}/c``
	* endpoint B : ``/a/b/c``
	* endpoint C : ``/a/{var1}/{var2}``

	- 输入 URI ： ``a/b/c`` 匹配 B ， 不会匹配 A 和 C ， 因数优先选择精确匹配
	- 输入 URI ： ``a/d/c`` 匹配 A， 变量 var=d ， 因为相对变量片段， 优先选择精确匹配。
	- 输入 URI ： ``a/x/y`` 匹配 C， 变量 var1=x, var2-y

#. 假设我们有两个 endpoint 
	
	* endpoint A: ``/{var1}/d``
	* endpoint B: ``/b/{var2}``

	输入 URL ： ``/b/d`` 匹配 B ， 变量 var2=d， 而不是 匹配 A。 因为匹配过程是从左至右。


当 URI　匹配不到　endpoint 时，实现不应该建立连接。[WSC-3.1.1-4]



3.1.2 子协议协商
^^^^^^^^^^^^^^^^^^

在服务器端创建时， 必须提供一个具有优先顺序支持协议列表。
在子协议协商时， 配置检查客户端提供的子协议列表， 
并从自己的列表中选择第一个存在客户端提供的列表中的子协议，
如果都不匹配， 则没有选中。[WSC-3.1.2-1]

3.1.3 扩展修改
^^^^^^^^^^^^^^^

在一个打开的握手中， 客户端提供一个希望使用的扩展列表。
服务器端的默认配置从该列表中选择能够支持的扩展， 并按请求的顺序放置了。[WSC-3.1.3-1]

3.1.4 来源检查
^^^^^^^^^^^^^^^

服务器端默认配置通过来源头部信息检查主机名， 
如果不是一个有效的主机名， 握手过程将失败。[WSC-3.1.4-1]

3.1.5 Handshake Modiﬁcation

服务器端对握手过程不做如上所述以外的修改。 [WSC-3.1.5-1]

开发者可能希望自定义以上所述的配置和握手协商策略，
那么， 他们必自提供一个自已的 ``ServerEndpointConﬁg.Conﬁgurator`` 实现。

比如，开发者可能希望更多地干预握手过程。
他们希望用 cookies 来追踪客户端， 
或在握手响应中插入一些应用特有的头部信息。
那么， 他们需要在 ``ServerEndpointConﬁg.Conﬁgurator`` 上实现 ``modifyHandshake()`` 方法，
在这个方法中， 他们有完整的权限访问握手过程中的 ``HandshakeRequest`` 和 ``HandshakeResponse`` 。


3.1.6 自定义状态或处理跨服务器的 Endpoint 实例
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The developer may also implement ServerEndpointConﬁg.Conﬁgurator in order to hold custom application state or methods for other kinds of application speciﬁc processing that is accessible from all Endpoint instances of the same logical endpoint via the EndpointConﬁg object.

（这句太难翻译了。。囧）


3.1.7 创建 Endpoint 自定义
^^^^^^^^^^^^^^^^^^^^^^^^^^^

开发者可以通过提供一个重写了 ``getEndpointInstance()`` 方法的 ``ServerEndpointConﬁg.Conﬁgurator`` 来控制 Endpoint 的创建。
当新的客户端连接逻辑endpoint 时， 实现都必须调用该方法。[WSC-3.1.7-1] 
平台上这个方法的默认实现是每次该方法被调用时， 
都返回 endpoint 类的新实例。[WSC-3.1.7-2]

这样， 开发者可以使客户端连接到逻辑 endpoint 时， 只有对应一个 endpoint 类的实例。
在这种情况下， 开发者必须注意在这个单例上发生的并发情况。
比如， 两个不同的端点同时发送了一个消息。

3.2 客户端配置
------------------------

为了连接 websocket 客户端和服务器端， 实现需要一些配置信息。
除了编码器(encoders)和解码器(decoders)， Java WebSocket API 需要以下属性:

3.2.1 子协议(Subprotocols)
^^^^^^^^^^^^^^^^^^^^^^^^^^

默认客户端配置按开发者提供的顺序发送子协议列表。
子协议名称会在握手过程中使用。


3.2.2 扩展（Extensions）
^^^^^^^^^^^^^^^^^^^^^^^^^

默认客户端配置按开发者提供的顺序发送扩展列表，
扩展及参数将在握手过程中使用。
[WSC-3.2.2-1]


3.2.3 服务器端配置修改
^^^^^^^^^^^^^^^^^^^^^^^

一些客户端希望适配客户端及服务器端握手的交互过程。
开发者可以通过提供自己的 ``ClientEndpointConﬁg.Conﬁgurator`` 实现来重写默认实现的行为， 
以满足应用程序的特定需求。