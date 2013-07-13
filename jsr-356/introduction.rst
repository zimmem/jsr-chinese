第一章 介绍
===========

本规范定义了一套开发 websocket 应用的 java 接口。 读者必须熟悉 WebSocket 协议。 WebSocket 协议是 HTML5 的一部分，该协议承诺更容易开发交互式 Web 应用， 且带来更高的网络效率。 更多关于 WebSocket 的信息， 请参考：

- The WebSocket Protocol speciﬁcation [1]
- The WebSocket API for JavaScript [2]

1.1 本文档目的
------------------

本文档同时做为 Java WebSocket 的 API 文档及规范。 本规范定义了一个 Java WebSocket API 必须满足的要求。 本规范基于 Java Community Process 的规则开发。 本规范同时提供了测试套件及一个参考实现， 参考实现满足规范要求， 且通过所有的测试。 本规范定义了开发 Java WebSocket 应用的标准。 文档中有很多关于使用 Java WebSocket API 的信息， 但本文档的目的并不是成为一份开发指南。 同样，如果你想开发一个 WebSocket API 实现， 文档中有很多有利信息， 但这份文档的目的并不是指导开发者如何实现所有的要求特性。

1.2 本规范目标
-------------

本规范的目标是定义一套在 Java 平台开发支持 WebSocket API 容器的要求。 尽管这个文档对使用 WebSocket API 的开发者有很多有用的信息。但这个文档不是一份开发指南。

1.3 规范中使用到的术语
-----------------------

- **endpoint** 一个 WebSocket endpoint 是一个代表通过 Websocket 连接的两端之一的 Java 组件。
- **connection** 一个 WebSocket connection 指两个 endpoint 通过 WebSocket 协议连接的网络链接。
- **peer** 在 WebSocket endpoint 的上下文中使用， 用于表示当前 endpoint 通过 WebSocket connection 连接的另一端。
- **session** WebSocket session 用于表示两个通信端点的 WebSocket 序列。client endpoints and server endpoints 客户端 endpoint 指的是启动一个新连接但不会应答（accept) 另一端点的 endpoint. 服务器端 endpoint 是指应答（accept）WebSocket 连接但不会主动启动对其它端点连接的 endpoing.


1.4 规范约定
-------------

文档按照 RFC 2119 [3] 规范使用关键词 “必须”（MUST）， “绝对不”（MUST NOT）， “可以”（SHALL）， “不可以”，
“应该”（SHOULD）， “不应该”（SHOULD NOT）“建议”（RECOMENDED）， “可以”（MAY）， “建议”（OPTIONAL）来描述解释。
（译文将在这些语气词后的括号中标英文原文）。

另外， 规范规定的要求将使用 WSC (WebSocket Compatibility) 加上一个数字来表示某个要求可以使用哪个一致性测试套件来做识别，
比如： WSC-12

Java 代码及数据片段样本将按照如下格式展示::

	 package com.example.hello;

	 public class Hello {
		 public static void main(String args[]) {
		 	System.out.println("Hello World");
		 }
	 }

URI 表示为 “http://example.org/...”的表示， 同样 “http://example.com/...” 也表示应用程序或上下文相关的 URI。

本文档除了示例， 备注和明确标注为 “非规范” 的部分， 其它部分保证规范性。 非规范的备注以如下形式展示。

**备注** *这是一个备注*

1.5 专家组成员
---------------

本规范由Java Community Process开发，做为 JSR 356 [4] 的一部分。
是 JSR 356 专家组所有成员的共同成果。 可以在参考条目[5]找到完整的邮件归档。
专家组成员如下：

- Jean-Francois Arcand (Individual Member)
- Greg Wilkins (Intalio)
- Scott Ferguson (Caucho Technology, Inc)
- Joe Walnes (DRW Holdings, LLC)
- Minehiko IIDA (Fujitsu Limited)
- Wenbo Zhu (Google Inc.)
- Bill Wigger (IBM)
- Justin Lee (Individual Member)
- Danny Coward (Oracle)
- Rmy Maucherat (RedHat)
- Moon Namkoong (TmaxSoft, Inc.)
- Mark Thomas (VMware)
- Wei Chen (Voxeo Corporation)
- Rossen Stoyanchev (VMware)

1.6 致谢
--------

在开发本规范过程中收到了很多评论， 反馈及建议。
尤其感谢 Jitendra Kotamraju, Martin Matula, Stˇ epˇ an Kop ´ ˇriva, Pavel Bucek, Dhiru Panday, Jondean Healey, Joakim Erdfelt, Dianne Jiao, Michal Conos, Jan Supol.

参考条目
------------

[1] I. Fette and A. Melnikov. RFC 6455: The WebSocket Protocol. RFC, IETF, December 2011. See http://www.ietf.org/rfc/rfc6455.txt.

[2] Ian Hickson. The WebSocket API. Note, W3C, December 2012. See http://dev.w3.org/html5/websockets/.
