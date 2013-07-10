---
layout: post
title : Java™ API for WebSocket
---

{{ post.title }}
================

<div class="jsr-version">
	Version 1.0 Final
	April 26th, 2013
</div>

<div class="jsr-comment-to">
	Danny Coward
	Comments to: users@websocket-spec.java.net
</div>

<div class="jsr-corporation">
	Oracle Corporation
	500 Oracle Parkway, Redwood Shores, CA 94065 USA.
</div>

第一章 介绍
-----------

本规范定义了一套开发 websocket 应用的 java 接口。 
读者必须熟悉 WebSocket 协议。
WebSocket 协议是 HTML5 的一部分，该协议承诺更容易开发交互式 Web 应用， 且带来更高的网络效率。
更多关于 WebSocket 的信息， 请参考：

* The WebSocket Protocol speciﬁcation [1]
* The WebSocket API for JavaScript [2]

### 本文档目的

本文档同时做为 Java WebSocket 的 API 文档及规范。
本规范定义了一个 Java WebSocket API 必须满足的要求。
本规范基于  Java Community Process 的规则开发。
本规范同时提供了测试套件及一个参考实现， 参考实现满足规范要求， 且通过所有的测试。
本规范定义了开发 Java WebSocket 应用的标准。
文档中有很多关于使用 Java WebSocket API 的信息， 但本文档的目的并不是成为一份开发指南。
同样，如果你想开发一个 WebSocket API 实现， 文档中有很多有利信息，
但这份文档的目的并不是指导开发者如何实现所有的要求特性。

### 本规范目标

本规范的目标是定义一套在 Java 平台开发支持 WebSocket API  容器的要求。
尽管这个文档对使用 WebSocket API 的开发者有很多有用的信息。但这个文档不是一份开发指南。

### 规范中使用到的术语

* **endpoint**  一个 WebSocket endpoint 是一个代表通过 Websocket 连接的两端之一的 Java 组件。
* **connection** 一个 WebSocket connection 指两个 endpoint 通过 WebSocket 协议连接的网络链接。	
* **peer** 在 WebSocket endpoint 的上下文中使用， 用于表示当前 endpoint 通过 WebSocket connection 连接的另一端。
* **session** WebSocket session 用于表示两个通信端点的 WebSocket 序列。
* **client endpoints and server endpoints** 客户端 endpoint 指的是启动一个新连接但不会应答（accept) 另一端点的 endpoint. 服务器端 endpoint 是指应答（accept）WebSocket 连接但不会主动启动对其它端点连接的 endpoing.

参考条目
------
[1] I. Fette and A. Melnikov. RFC 6455: The WebSocket Protocol. RFC, IETF, December 2011. See
http://www.ietf.org/rfc/rfc6455.txt.

[2] Ian Hickson. The WebSocket API. Note, W3C, December 2012. See
http://dev.w3.org/html5/websockets/.	