---
layout: post
title: "SIP Headers"
date: 2017-04-23 19:45:06 +0800
comments: true
categories: protocol sip
---

##引言
SIP协议的设计很多参考了已有的HTTP协议，例如都是用的文本，验证用户相关的header和content-type和body的设计。这里主要说一下SIP独有的这些header，其中对路由相关的header又是重点要说的。

##From和To
这两个header是作为标识存在的，对协议本身的路由来说不起什么作用。在实际电话呼叫中会标识主叫方和被叫方。其中的tag参数被选用来标识session。除了这两个tag参数外，session的标识还要用到另外一个header： Call-ID的值。下面是[RFC3261](https://tools.ietf.org/html/rfc3261)里面的一个例子：
{% codeblock %}
From: "Bob" <sips:bob@biloxi.com> ;tag=a48s
From: sip:+12125551212@phone2net.com;tag=887s
From: Anonymous <sip:c8oqz84zk7z@privacy.org>;tag=hyh8

To: Carol <sip:carol@chicago.com>
{% endcodeblock %}
可以看到这两个header里可以包含一个用来描述这个URI的display name。在呼叫中这个就会用来标识呼叫的主被叫的名字。

From和To在一侧的请求和响应里是一致的，但是和呼叫另一侧发过来的请求是相反的。

##Contact
这个header用来标识session内这个UA期望对方request发送到的地址。在一个Invite发起的session中，一般会包含在Invite和对应的响应里。它所表示的地址就是session内ACK，Bye，re-Invite等发送到这个UA的地址。
{% codeblock %}
Contact: <sip:pc33.atlanta.com>
{% endcodeblock %}

##Via
上面讲的Contact是为request指示地址，而Via是为request对应的response来指示地址的。它是在request被发送的时候添加的，而对方收到请求之后会把响应发给这个header标示的地址。如果涉及到NAT或者防火墙之类的，这个发送方添加的地址和接收方收到的源地址不同的情况。这种情况下协议添加了个received的参数来标示实际收到的request是从哪发来的。这样保证了response能原路发送回request发送方。RFC3261中介绍了这种情况下的一个例子：
{% codeblock %}
INVITE sip:bob@Biloxi.com SIP/2.0
Via: SIP/2.0/UDP bobspc.biloxi.com:5060
{% endcodeblock %}
这个request的接收方发现它的源地址是和via里面的地址不一样的，所以就在里面加上了received参数来标示实际的源地址：
{% codeblock %}
INVITE sip:bob@Biloxi.com SIP/2.0
Via: SIP/2.0/UDP bobspc.biloxi.com:5060;received=192.0.2.4
{% endcodeblock %}

Via里面还有个重要的参数branch来标示这个request对应的transaction。

##Request-Uri
这个准确来说不是一个header，因为它是request第一行里请求类型后面的URI。它是用来标识请求要发向的地址的，因为我们前面说了To header不决定路由。

##Route和Record-Route
在没有Proxy的情况下只要一个Request-Uri就能把request路由到目的地了，但是实际情况是request总要经过一些Proxy来达到目的地。所以我们还要用Route来标示请求中间经过的节点。当Proxy收到请求后发现top Route是自己的地址就会去掉转发给下一个目的地。如果这个Proxy配置成Record Route的，就会在请求中加入Record-Route这个header，这个header的加入就表示session内的其他请求也会经过这个节点。所以Record-Route是保存在session内的状态，它和Contact共同决定了session内请求的路由路径。


{% codeblock Simple Proxy %}
+---------+              +-----------+              +-----------+
|         |              |           |              |           |
|UA-Caller|              |  Proxy    |              | UA-Callee |
+----+----+              +-----+-----+              +-----+-----+
     |                         |                          |
     |                         |                          |
     |                         |                          |
     |                         |                          |
     +------------------------->                          |
     |INVITE sip:callee@192.168.1.7                       |
     |Route: sip:192.168.1.5                              |
     |                         |                          |
     |                         |                          |
     |                         |                          |
     |                         +-------------------------->
     |                         |    INVITE sip:callee@192.168.1.7
     |                         |    Record-Route: sip:192.168.1.5
     |                         |                          |
     |                         |                          |
     |                         |                          |
     |                         |                          |
     |                         <--------------------------+
     |                         |   SIP/2.0 200 OK         |
     |                         |   Record-Route: sip:192.168.1.5
     |                         |                          |
     |                         |                          |
     <-------------------------+                          |
     |   SIP/2.0 200OK         |                          |
     |   Record-Route: sip:192.168.1.5                    |
     |                         +                          |
     |                         |                          |
     |                         |                          |
     |                         |                          |
     |                         |                          |
     |                         |                          |
     +------------------------->                          |
     |   BYE sip:callee@192.168.1.5                       |
     |   Route: sip:192.168.1.5|                          |
     |                         |                          |
     |                         |                          |
     |                         +-------------------------->
     |                         |   BYE sip:callee@192.168.1.5
     |                         |                          |
     |                         |                          |
     |                         |                          |
     +                         +                          +
{% endcodeblock %}
这里的Record-Route通过INVITE传给了UA-Callee，并通过200OK返回给了UA-Caller，保证了后续的BYE（UA-Callee发起的也是一样）必须经过Proxy才能到对方。

如果request需要经过多个Proxy，就有可能会有多个Record-Route。这个Record-Route的添加也是按顺序添加到request里的，这样保证消息能够按初始request的路径来路由。

如果这个Proxy正好部署在网络边界上（多网卡机器），有可能UA-Caller这侧是一个网段而UA-Callee成了另一个网段，或者两侧使用的transport不同。这样由于Record-Route是共享给两侧使用的，这就会造成一侧的消息发不到Proxy。因为Record-Route没有像Via那样可以加个received的参数来揭示源地址。

[RFC5658](https://tools.ietf.org/html/rfc5658)通过添加两个Record-Route来巧妙的解决了这个问题。可以直接参考那个RFC里面的这个例子：

```
       UA1              Proxy "P1"               UA2
      (IPv4)            (IPv4/IPv6)             (IPv6)
        |                    |                    |
        |   F1 INVITE        |                    |
        |------------------->|      F2 INVITE     |
        |                    |------------------->|
        |    100 Trying      |                    |
        |<-------------------|                    |
        |                    |    F3 200 OK       |
        |    F4 200 OK       |<-------------------|
        |<-------------------|                    |
        |                    |                    |
        |       F5 ACK       |                    |
        |------------------->|       F6 ACK       |
        |                    |------------------->|
        |                    |                    |
        |                    |        F7 BYE      |
        |       F8 BYE       |<-------------------|
        |<-------------------|                    |

            Figure 2: IPv4 to IPv6 Multi-Homed Proxy Illustration


   F1 INVITE UA1 -> P1 (192.0.2.254:5060)

   INVITE sip:bob@biloxi.example.com SIP/2.0
   Route: <sip:192.0.2.254:5060;lr>
   From: Alice <sip:alice@atlanta.example.com>;tag=1234
   To: Bob <sip:bob@biloxi.example.com>
   Contact: <sip:alice@192.0.2.1>

           F2 INVITE P1 (2001:db8::1) -> UA2

           INVITE sip:bob@biloxi.example.com SIP/2.0
           Record-Route: <sip:[2001:db8::1];lr>
           Record-Route: <sip:192.0.2.254:5060;lr>
           From: Alice <sip:alice@atlanta.example.com>;tag=1234
           To: Bob <sip:bob@biloxi.example.com>
           Contact: <sip:alice@192.0.2.1>

                   Dialog State at UA2:
                   Local URI     = sip:bob@biloxi.example.com
                   Remote URI    = sip:alice@atlanta.example.com
                   Remote target = sip:alice@192.0.2.1
                   Route Set     = sip:[2001:db8::1];lr
                                   sip:192.0.2.254:5060:lr


                   F3 200 OK UA2 -> P1 (2001:db8::1)

                   SIP/2.0 200 OK
                   Record-Route: <sip:[2001:db8::1];lr>
                   Record-Route: <sip:192.0.2.254:5060;lr>
                   From: Alice <sip:alice@atlanta.example.com>;tag=1234
                   To: Bob <sip:bob@biloxi.example.com>;tag=4567
                   Contact: <sip:bob@[2001:db8::33]>

           F4 200 OK P1 -> UA1

           SIP/2.0 200 OK
           Record-Route: <sip:[2001:db8::1];lr>
           Record-Route: <sip:192.0.2.254:5060;lr>
           From: Alice <sip:alice@atlanta.example.com>;tag=1234
           To: Bob <sip:bob@biloxi.example.com>;tag=4567
           Contact: <sip:bob@[2001:db8::33]>

   Dialog State at UA1:
   Local URI     = sip:alice@atlanta.example.com
   Remote URI    = sip:bob@biloxi.example.com
   Remote target = sip:bob@[2001:db8::33]
   Route Set     = sip:192.0.2.254:5060:lr
                   sip:[2001:db8::1];lr

   F5 ACK UA1 -> P1 (192.0.2.254:5060)

   ACK sip:bob@[2001:db8::33] SIP/2.0
   Route: <sip:192.0.2.254:5060:lr>
   Route: <sip:[2001:db8::1];lr>
   From: Alice <sip:alice@atlanta.example.com>;tag=1234
   To: Bob <sip:bob@biloxi.example.com>;tag=4567

           F6 ACK P1 (2001:db8::1) -> UA2

           ACK sip:bob@[2001:db8::33] SIP/2.0
           From: Alice <sip:alice@atlanta.example.com>;tag=1234
           To: Bob <sip:bob@biloxi.example.com>;tag=4567
           (both Route headers have been removed by the proxy)

                   F7 BYE UA2 -> P1 (2001:db8::1)

                   BYE sip:alice@192.0.2.1 SIP/2.0
                   Route: <sip:[2001:db8::1];lr>
                   Route: <sip:192.0.2.254:5060:lr>
                   From: Bob <sip:bob@biloxi.example.com>;tag=4567
                   To: Alice <sip:alice@atlanta.example.com>;tag=1234


           F8 BYE P1 (192.0.2.254:5060) -> UA1

           BYE sip:alice@192.0.2.1 SIP/2.0
           From: Bob <sip:bob@biloxi.example.com>;tag=4567
           To: Alice <sip:alice@atlanta.example.com>;tag=1234

```
里面是用的ipv4和ipv6举的例子，不同网段或者transport的是类似的。


##CSeq
这个表示了会话(session)内的消息的顺序，呼叫两侧的CSeq号是分别维护的。

好了这篇到这告一段落了。其他的header比较简单了，可以查看RFC得到明确的意义。
