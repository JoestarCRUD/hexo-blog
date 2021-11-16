---
title: ZMQ(3.2)官网文档翻译 第一章——基础知识
date: 2021-11-14 23:11:20
categories: 
- 消息队列
- 中间件
tags:
 - ZeroMQ
 - 官方文档翻译
---

## Fixing the World

&emsp;&emsp;如何理解ZeroMQ？我们中的一些人首先会开始输出他们的溢美之词。*如类固醇上的官能团，如同自带路由功能的邮箱，快如闪电！*其他人试图分享他们的启蒙时刻，当一切变得显而易见时，那个 zap-pow-kaboom satori 范式转变时刻(对不起译者实在对生物医药一窍不通，这是一个试图让本就复杂的概念更复杂的糟糕比喻orz)。 *事情变得简单了。复杂性消失了。它打开了思路。* 其他人试图通过比较来解释。 *它更小、更简单，但看起来仍然很熟悉。* 就我个人而言，我更愿意忆起我们创作ZMQ的初衷，因为这很可能是你，读者，今天仍然在困惑的烦恼。

&emsp;&emsp;编程是被伪装成了艺术的科学，因为我们大多数人都不了解软件的物理原理，而且很少（如果有的话）教过它。软件的物理学不是算法、数据结构、语言和抽象。这些只是我们制造、使用和扬弃的工具。软件的真正物理学是人的物理学——特别是我们在复杂性方面的局限性，以及我们共同努力解决大问题的愿望。这就是编程科学：编写人们可以*轻松*理解和使用的模块，人们将共同努力解决最大的问题。

&emsp;&emsp;我们生活在一个互联的世界中，现代软件必须引导这个世界。因此，未来最大的解决方案的模块是互联互通和大规模并行。代码仅仅是“强大而安静”是不够的。代码必须与代码对话。代码必须健谈、善于交际、关系密切。代码必须像人脑一样运行，数以万亿计的单个神经元相互发送消息，一个去中心化的大规模并行网络，没有单点故障，但能够解决极其困难的问题。代码的未来看起来像人脑也绝非偶然，因为每个网络的终端，在某种意义上说，就是人脑。

&emsp;&emsp;如果您在工作中接触过线程、协议或网络，您就会意识到这是一个impossible mission，如梦似幻。当您开始处理现实生活中的情况时，即使通过几个套接字连接几个程序也是非常讨厌的。万亿？代价将是难以想象的。连接计算机是如此困难，以至于实现此目的的软件和服务都是一项价值数十亿美元的生意。

&emsp;&emsp;因此，我们生活在一个代码编写胜过我们使用能力的时代。我们在 1980 年代遇到了软件危机，当时像 Fred Brooks 这样的领先软件工程师认为没有“银弹”可以“保证在生产力、可靠性或简单性方面提高哪怕一个数量级”。

&emsp;&emsp;Brooks错过了免费和开源软件，它解决了这场危机，使我们能够有效地共享知识。今天，我们面临另一场软件危机，但我们很少谈论它。只有最大、最富有的公司才能负担得起创建互联应用程序的费用。我们的确有云服务，但它是专有的。我们的数据和知识正在从我们的个人计算机中消失到我们无法访问且无法与之竞争的云中。谁拥有我们的社交网络？这仿佛反向革命回了大型机年代。

&emsp;&emsp;我可以把政治哲学的部分留给我的下一本书。关键是虽然互联网提供了大规模连接代码的潜力，但现实情况是，这对我们大多数人来说是遥不可及的，因此还有很多有趣的问题（在健康、教育、经济、交通等方面）仍无法得到解决因为无法我们无法连接代码，因此无法把可以共同解决这些问题的大脑连接起来。

&emsp;&emsp;已经有很多尝试来解决连接代码的挑战。有数以千计的 IETF 规范，每一个都解决了难题的一部分。对于开发者来说，HTTP 可能是一种简单能用的解决方案，但它可以说是恶化了问题，因为它鼓励开发人员和架构师去考虑巨大复杂的服务端，同时却把客户端认为是瘦弱愚蠢的。

&emsp;&emsp;所以今天人们仍然使用原始 UDP 和 TCP、专有协议、HTTP 和 Websockets 连接应用程序。它仍然痛苦、缓慢、难以扩展，并且基本上是中心化的。分布式 P2P 架构主要是用来玩的，不是用来解决问题的。有多少应用程序使用 Skype 或 Bittorrent 来交换数据？

&emsp;&emsp;这让我们回到了编程科学。为了修复这个世界，我们需要做两件事。一，解决“如何在任何地方将任何代码连接到任何代码”的普适性问题。第二，将最简单易构的模块包装起来让人们可以*轻松*理解和使用。这听起来简单得可笑。也许是这样。但这就是重点。

## 开始假设

&emsp;&emsp;我们假设您至少使用了 ZeroMQ 3.2 版。我们假设您使用的是 Linux 机器或类似的东西。我们假设您或多或少可以阅读 C 代码(译者帮您选择了，您懂Java，对的，就这么独断)，因为这是(译者觉得恐怕不是 &#x1F436;)示例的默认语言。我们假设当我们编写像 PUSH 或 SUBSCRIBE 这样的常量时，我们假设如果编程语言需要的话，您就能够看懂 ZMQ_PUSH 或 ZMQ_SUBSCRIBE这样的命名或编码。

## 获取示例

&emsp;&emsp;示例位于公共 GitHub 仓库中。获取所有示例的最简单方法是clone 这个仓库：

```shell
git clone --depth=1 https://github.com/imatix/zguide.git
```

接下来，浏览示例子目录。您会按语言找到示例。如果您使用的语言缺少示例，我们鼓励您提交翻译。许多人的共同努力，就是本文变得如此有用的原因。所有示例均在 MIT/X11 下获得许可。

## 寻找，就寻见

&emsp;&emsp;让我们开始把。遵照国际惯例我们当然从一个 Hello World 示例开始。我们将编写一个客户端和一个服务端。客户端向服务器发送“Hello”，服务器回复“World”。这是 Java语言中的服务器，它在端口 5555 上打开一个 ZeroMQ 套接字，读取其上的请求，并对每个请求回复“World”：

```java
package guide;

//
//  Hello World server in Java
//  Binds REP socket to tcp://*:5555
//  Expects "Hello" from client, replies with "World"
//

import org.zeromq.SocketType;
import org.zeromq.ZMQ;
import org.zeromq.ZContext;

public class hwserver
{
    public static void main(String[] args) throws Exception
    {
        try (ZContext context = new ZContext()) {
            // Socket to talk to clients
            ZMQ.Socket socket = context.createSocket(SocketType.REP);
            socket.bind("tcp://*:5555");

            while (!Thread.currentThread().isInterrupted()) {
                byte[] reply = socket.recv(0);
                System.out.println(
                    "Received " + ": [" + new String(reply, ZMQ.CHARSET) + "]"
                );

                Thread.sleep(1000); //  Do some 'work'

                String response = "world";
                socket.send(response.getBytes(ZMQ.CHARSET), 0);
            }
        }
    }
}
```

**Figure 2 - Request-Reply**
{% asset_img fig2.png Figure 2 %}
&emsp;&emsp;REQ-REP 套接字对是同步的。客户端发出 [zmq_send()](http://api.zeromq.org/master:zmq-send) 然后 [zmq_recv()](http://api.zeromq.org/master:zmq-recv)，在一个循环中（或者一次，如果仅需一次的话）。执行任何其他序列（例如，连续发送两条消息）将导致 send 或 recv 调用的返回码为 -1。类似地，该服务按其需要的频率依次发出 [zmq_recv()](http://api.zeromq.org/master:zmq-recv) 和 [zmq_send()](http://api.zeromq.org/master:zmq-send)。

*文档给出了C，C++，PHP和Java的示例并比较异同，这部分译者就省略了，读者们有需要可以去官网阅读。*

这是客户端代码：

```java
package guide;

//
//  Hello World client in Java
//  Connects REQ socket to tcp://localhost:5555
//  Sends "Hello" to server, expects "World" back
//

import org.zeromq.SocketType;
import org.zeromq.ZMQ;
import org.zeromq.ZContext;

public class hwclient
{
    public static void main(String[] args)
    {
        try (ZContext context = new ZContext()) {
            //  Socket to talk to server
            System.out.println("Connecting to hello world server");

            ZMQ.Socket socket = context.createSocket(SocketType.REQ);
            socket.connect("tcp://localhost:5555");

            for (int requestNbr = 0; requestNbr != 10; requestNbr++) {
                String request = "Hello";
                System.out.println("Sending Hello " + requestNbr);
                socket.send(request.getBytes(ZMQ.CHARSET), 0);

                byte[] reply = socket.recv(0);
                System.out.println(
                    "Received " + new String(reply, ZMQ.CHARSET) + " " +
                    requestNbr
                );
            }
        }
    }
}
```

&emsp;&emsp;现在这看起来太简单了，不太现实，但正如我们已经了解到的，ZeroMQ 套接字具有超能力。您可以同时在此服务端上投放数千个客户端，并且它会继续愉快而快速地工作。为了好玩，尝试启动客户端，然后启动服务器，看看它仍将如何工作，然后思考一下这意味着什么。

&emsp;&emsp;让我们简要解释一下这两个程序实际上在做什么。他们创建了一个 ZeroMQ 上下文来使用，以及一个套接字。不要担心这些词是什么意思。你很快会弄懂的。服务器将其 REP（回复）套接字绑定到端口 5555。服务器在循环中等待请求，并且每次都以回复进行响应。客户端发送请求并从服务器读取回复。

&emsp;&emsp;如果你杀掉服务端进程 (Ctrl-C) 并重新启动它，客户端将无法正常恢复。从崩溃的进程中恢复并不是那么容易。制作一个可靠的请求-回复流非常复杂，我们会在Chapter 4 - Reliable Request-Reply Patterns.介绍它。

&emsp;&emsp;幕后发生了很多事情，但对我们程序员而言，重要的是代码的简短和优雅，以及即使在重负载下也不会崩溃的频率。这是请求-回复模式，可能是使用 ZeroMQ 的最简单方法。它类似于 RPC 和经典的C/S模型。

## 关于字符串的小注

&emsp;&emsp;除了以字节为单位的大小外，ZeroMQ 对您发送的数据一无所知。这意味着您有责任安全地格式化它，以便应用程序可以读取它。为对象和复杂数据类型执行此操作是协议缓冲区等专用库的工作。但即使是字符串，你也需要小心。

在 C 和其他一些语言中，字符串以空字节结尾。我们可以用额外的空字节发送一个像“HELLO”这样的字符串：

```c
zmq_send (requester, "Hello", 6, 0);
```

但是，如果您从另一种语言发送一个字符串，它可能不会包含该空字节。例如，当我们在 Python 中发送相同的字符串时，我们这样做：

```python
socket.send("Hello")
```

然后最终呈现的是长度（较短的字符串为一个字节）和字符串内容作为单个字符。

**Figure 3 - A ZeroMQ string**
{% asset_img fig3.png Figure 3 %}

&emsp;&emsp;如果你从一个 C 程序中读到这个，你会得到一些看起来像一个字符串的东西，并且可能偶然地表现得像一个字符串（如果幸运的是五个字节发现它们后面跟着一个无辜的空值），但不是一个恰当的字符串。当您的客户端和服务端不能对字符串格式达成一致时，您会得到奇怪的结果。

&emsp;&emsp;当您在 C 中从 ZeroMQ 接收字符串数据时，您根本无法相信它已安全终止。每次读取一个字符串时，都应该为一个额外的字节分配一个新的缓冲区空间，复制该字符串，并以空值正确终止它。

&emsp;&emsp;所以让我们建立一个规则，即 ZeroMQ 字符串是指定长度的，并且在没有尾随 null 的情况下在线发送。在最简单的情况下（我们将在我们的示例中这样做），一个 ZeroMQ 字符串巧妙地映射到一个 ZeroMQ 消息帧，它看起来像上图——一个长度和一些字节。

以下是我们在 C 中需要做的事情，以接收 ZeroMQ 字符串并将其作为有效的 C 字符串传递给应用程序：

```c
//  Receive ZeroMQ string from socket and convert into C string
//  Chops string at 255 chars, if it's longer
static char *
s_recv (void *socket) {
    char buffer [256];
    int size = zmq_recv (socket, buffer, 255, 0);
    if (size == -1)
        return NULL;
    if (size > 255)
        size = 255;
    buffer [size] = \0;
    /* use strndup(buffer, sizeof(buffer)-1) in *nix */
    return strdup (buffer);
}
```

&emsp;&emsp;这构成了一个方便的辅助函数，本着复用让我们有利可图的精神，让我们编写一个类似的 s_send 函数，以正确的 ZeroMQ 格式发送字符串，并将其打包成一个我们可以重用的头文件。其成果就是 zhelpers.h，它让我们可以用 C 编写更漂亮、更短的 ZeroMQ 应用程序。它是一个相当长的源代码，并且只对 C 开发人员有趣，所以请在闲暇时阅读。

## 关于命名约定的说明

&emsp;&emsp;zhelpers.h 中使用的前缀 s_ 和本指南中的示例是静态方法或变量的标识符。

## 版本报告

&emsp;&emsp;ZeroMQ 确实有多个版本，而且通常，如果您遇到问题，它会在以后的版本中修复。因此，准确了解您实际链接的是哪个版本的 ZeroMQ 是一个有用的技巧。下面是一个执行此操作的小程序：

```java
package guide;

import org.zeromq.ZMQ;

//  Report 0MQ version
public class version
{
    public static void main(String[] args)
    {
        String version = ZMQ.getVersionString();
        int fullVersion = ZMQ.getFullVersion();

        System.out.println(
            String.format(
                "Version string: %s, Version int: %d", version, fullVersion
            )
        );
    }
}
```

## 发送消息

&emsp;&emsp;第二种经典模式是单向数据分发，其中服务端将更新推送到一组客户端。让我们看一个例子，它推送由邮政编码、温度和相对湿度组成的天气更新。我们将生成随机值，就像真实的气象站一样。这是服务端，我们将为此应用程序使用端口 5556：

```java
package guide;

import java.util.Random;

import org.zeromq.SocketType;
import org.zeromq.ZMQ;
import org.zeromq.ZContext;

//
//  Weather update server in Java
//  Binds PUB socket to tcp://*:5556
//  Publishes random weather updates
//
public class wuserver
{
    public static void main(String[] args) throws Exception
    {
        //  Prepare our context and publisher
        try (ZContext context = new ZContext()) {
            ZMQ.Socket publisher = context.createSocket(SocketType.PUB);
            publisher.bind("tcp://*:5556");
            publisher.bind("ipc://weather");

            //  Initialize random number generator
            Random srandom = new Random(System.currentTimeMillis());
            while (!Thread.currentThread().isInterrupted()) {
                //  Get values that will fool the boss
                int zipcode, temperature, relhumidity;
                zipcode = 10000 + srandom.nextInt(10000);
                temperature = srandom.nextInt(215) - 80 + 1;
                relhumidity = srandom.nextInt(50) + 10 + 1;

                //  Send message to all subscribers
                String update = String.format(
                    "%05d %d %d", zipcode, temperature, relhumidity
                );
                publisher.send(update, 0);
            }
        }
    }
}
```

这种更新流没有开始也没有结束，就像一个永无止境的广播。

这是客户端应用程序，它监听更新流并获取与指定邮政编码有关的任何内容，默认情况下纽约市，因为这是开始任何冒险的好地方：

```java
package guide;

import java.util.StringTokenizer;

import org.zeromq.SocketType;
import org.zeromq.ZMQ;
import org.zeromq.ZContext;

//
//  Weather update client in Java
//  Connects SUB socket to tcp://localhost:5556
//  Collects weather updates and finds avg temp in zipcode
//
public class wuclient
{
    public static void main(String[] args)
    {
        try (ZContext context = new ZContext()) {
            //  Socket to talk to server
            System.out.println("Collecting updates from weather server");
            ZMQ.Socket subscriber = context.createSocket(SocketType.SUB);
            subscriber.connect("tcp://localhost:5556");

            //  Subscribe to zipcode, default is NYC, 10001
            String filter = (args.length > 0) ? args[0] : "10001 ";
            subscriber.subscribe(filter.getBytes(ZMQ.CHARSET));

            //  Process 100 updates
            int update_nbr;
            long total_temp = 0;
            for (update_nbr = 0; update_nbr < 100; update_nbr++) {
                //  Use trim to remove the tailing '0' character
                String string = subscriber.recvStr(0).trim();

                StringTokenizer sscanf = new StringTokenizer(string, " ");
                int zipcode = Integer.valueOf(sscanf.nextToken());
                int temperature = Integer.valueOf(sscanf.nextToken());
                int relhumidity = Integer.valueOf(sscanf.nextToken());

                total_temp += temperature;
            }

            System.out.println(
                String.format(
                    "Average temperature for zipcode '%s' was %d.",
                    filter,
                    (int)(total_temp / update_nbr)
                )
            );
        }
    }
}
```

**Figure 4 - Publish-Subscribe**
{% asset_img fig4.png Figure 4 %}

请注意，当您使用 SUB 套接字时，您必须使用 zmq_setsockopt() 和 SUBSCRIBE 设置订阅，如此代码所示。如果您不设置任何订阅，您将不会收到任何消息。对于初学者来说，这是一个常见的错误。订阅者可以设置多个订阅加在一起。也就是说，如果更新与任何订阅匹配，订阅者就会收到它。订阅者还可以取消特定订阅。订阅通常（但不总是）是可打印的字符串。请参阅 [zmq-setsockopt](http://api.zeromq.org/master:zmq-setsockopt) 了解其工作原理。

PUB-SUB 套接字对是异步的。客户端执行 [zmq-recv()](http://api.zeromq.org/master:zmq-recv)，循环执行（或者一次，如果这就是它所需要的）。尝试向 SUB 套接字发送消息将导致错误。类似地，该服务会根据需要经常执行 [zmq_send()](http://api.zeromq.org/master:zmq-send)，但不得在 PUB 套接字上执行 [zmq-recv()](http://api.zeromq.org/master:zmq-recv)。

理论上，对于 ZeroMQ 套接字，哪一端连接和哪一端绑定并不重要。但是，在实践中存在未记录的差异，我稍后会谈到。现在，绑定 PUB 并连接 SUB，除非您的网络设计使之成为不可能。

关于 PUB-SUB 套接字还有一件更重要的事情需要了解：您无法准确地知道订阅者何时开始接收消息。即使你启动了一个订阅者，等待一段时间，然后再启动发布者，**订阅者总是会错过发布者发送的第一条消息**。这是因为当订阅者连接到发布者时（花费很少但非零的时间），发布者可能已经在向外发送消息。

这种 “slow joiner” 的问题经常会困扰很多的人，我们将详细解释它。请记住，ZeroMQ 在后台进行异步 I/O。假设您有两个节点按以下顺序执行此操作：

+ 订阅者连接到端点并接收和计算消息。
+ 发布者绑定到端点并立即发送 1,000 条消息。

那么订阅者很可能不会收到任何东西。你会头疼，检查你是否设置了正确的过滤器，然后再试一次，订阅者仍然不会收到任何东西。

建立 TCP 连接涉及到握手和结束握手，这需要几毫秒，具体取决于您的网络和对等点之间的hops。那个时候，ZeroMQ 可以发送很多消息。为了论证起见，假设建立连接需要 5 毫秒，并且同一链接每秒可以处理 1M 消息。在订阅者连接到发布者的 5 毫秒期间，发布者只需要 1 毫秒即可发送这些 1K 消息。

在Chapter 2 - Sockets and Patterns 中，我们将解释如何同步发布者和订阅者，以便在订阅者真正连接并准备好之前不会开始发布数据。有一个简单而愚蠢的方法来延迟发布者，那就是sleep。但是，不要在实际应用程序中这样做，因为它非常脆弱，而且不优雅和并且缓慢。使用 sleep 向自己证明发生了什么，然后阅读Chapter 2 - Sockets and Patterns，看看如何正确地解决这一问题。

同步的替代方法是简单地假设发布的数据流是无限的，没有开始也没有结束。还假设订阅者不在乎启动之前发生了什么。这就是我们构建天气客户端示例的方式。

因此，客户端订阅其选择的邮政编码并收集该邮政编码的 100 个更新。如果邮政编码是随机分布的，这意味着来自服务器的大约 1000 万次更新。您可以启动客户端，然后启动服务端，客户端将继续工作。您可以根据需要随时停止和重新启动服务端，客户端将继续工作。当客户端收集了它的一百个更新时，它计算平均值，打印它，然后退出。

关于发布-订阅（pub-sub）模式的一些要点：

+ 订阅者可以连接到多个发布者，每次使用一个连接调用。然后数据将到达并被交错（公平队列），因此没有一个发布者会淹没其他发布者。
+ 如果发布者没有连接的订阅者，那么它只会丢弃所有消息。
+ 如果您使用 TCP 并且订阅者很慢，消息将在发布者上排队。稍后我们将研究如何使用“high-water mark”来保护发布者免受此类攻击。
+ 从 ZeroMQ v3.x 开始，当使用连接的协议（tcp:@<>@ 或 ipc:@<>@）时，过滤发生在发布者端。使用 epgm:@<//>@ 协议，过滤发生在订阅者端。在 ZeroMQ v2.x 中，所有过滤都发生在订阅者端。

这是在我的笔记本电脑上接收和过滤 10M 消息所需的时间，这是 2011 年时代的英特尔 i5，性能不错但没什么特别的：

```shell
$ time wuclient
Collecting updates from weather server...
Average temperature for zipcode '10001 ' was 28F

real    0m4.470s
user    0m0.000s
sys     0m0.008s
```

## 分而治之

**Figure 5 - Parallel Pipeline**
{% asset_img fig5.png Figure 5 %}

作为最后一个例子（你肯定已经厌倦了代码并想重新深入研究关于比较抽象规范的语言学讨论），让我们做一点超级计算。然后再喝咖啡。我们的超级计算应用程序是一个相当典型的并行处理模型。我们有：

+ 产生可以并行完成的任务的ventilator
+ 一组处理任务的workers
+ 从工作进程收集结果的sink

实际上，workers在超快的机器上运行，也许使用 GPU（图形处理单元）来进行困难的数学运算。这是ventilator。它生成 100 个任务，每个任务都有一条消息告诉workers睡眠几毫秒：

```java
package guide;

import java.util.Random;

import org.zeromq.SocketType;
import org.zeromq.ZMQ;
import org.zeromq.ZContext;

//
//  Task ventilator in Java
//  Binds PUSH socket to tcp://localhost:5557
//  Sends batch of tasks to workers via that socket
//
public class taskvent
{
    public static void main(String[] args) throws Exception
    {
        try (ZContext context = new ZContext()) {
            //  Socket to send messages on
            ZMQ.Socket sender = context.createSocket(SocketType.PUSH);
            sender.bind("tcp://*:5557");

            //  Socket to send messages on
            ZMQ.Socket sink = context.createSocket(SocketType.PUSH);
            sink.connect("tcp://localhost:5558");

            System.out.println("Press Enter when the workers are ready: ");
            System.in.read();
            System.out.println("Sending tasks to workers\n");

            //  The first message is "0" and signals start of batch
            sink.send("0", 0);

            //  Initialize random number generator
            Random srandom = new Random(System.currentTimeMillis());

            //  Send 100 tasks
            int task_nbr;
            int total_msec = 0; //  Total expected cost in msecs
            for (task_nbr = 0; task_nbr < 100; task_nbr++) {
                int workload;
                //  Random workload from 1 to 100msecs
                workload = srandom.nextInt(100) + 1;
                total_msec += workload;
                System.out.print(workload + ".");
                String string = String.format("%d", workload);
                sender.send(string, 0);
            }
            System.out.println("Total expected cost: " + total_msec + " msec");
            Thread.sleep(1000); //  Give 0MQ time to deliver
        }
    }
}
```

这是worker 应用程序。它接收到一条消息，休眠指定的秒数，然后发出它已完成的信号：

```java
package guide;

import org.zeromq.SocketType;
import org.zeromq.ZMQ;
import org.zeromq.ZContext;

//
//  Task worker in Java
//  Connects PULL socket to tcp://localhost:5557
//  Collects workloads from ventilator via that socket
//  Connects PUSH socket to tcp://localhost:5558
//  Sends results to sink via that socket
//
public class taskwork
{
    public static void main(String[] args) throws Exception
    {
        try (ZContext context = new ZContext()) {
            //  Socket to receive messages on
            ZMQ.Socket receiver = context.createSocket(SocketType.PULL);
            receiver.connect("tcp://localhost:5557");

            //  Socket to send messages to
            ZMQ.Socket sender = context.createSocket(SocketType.PUSH);
            sender.connect("tcp://localhost:5558");

            //  Process tasks forever
            while (!Thread.currentThread().isInterrupted()) {
                String string = new String(receiver.recv(0), ZMQ.CHARSET).trim();
                long msec = Long.parseLong(string);
                //  Simple progress indicator for the viewer
                System.out.flush();
                System.out.print(string + '.');

                //  Do the work
                Thread.sleep(msec);

                //  Send results to sink
                sender.send(ZMQ.MESSAGE_SEPARATOR, 0);
            }
        }
    }
}
```

这是sink应用程序。它收集 100 个任务，然后计算整个处理花费的时间，因此我们可以确认如果有多个任务，我们可以确认workers 确实在并行运行：

```java
package guide;

import org.zeromq.SocketType;
import org.zeromq.ZMQ;
import org.zeromq.ZContext;

//
//  Task sink in Java
//  Binds PULL socket to tcp://localhost:5558
//  Collects results from workers via that socket
//
public class tasksink
{
    public static void main(String[] args) throws Exception
    {
        //  Prepare our context and socket
        try (ZContext context = new ZContext()) {
            ZMQ.Socket receiver = context.createSocket(SocketType.PULL);
            receiver.bind("tcp://*:5558");

            //  Wait for start of batch
            String string = new String(receiver.recv(0), ZMQ.CHARSET);

            //  Start our clock now
            long tstart = System.currentTimeMillis();

            //  Process 100 confirmations
            int task_nbr;
            int total_msec = 0; //  Total calculated cost in msecs
            for (task_nbr = 0; task_nbr < 100; task_nbr++) {
                string = new String(receiver.recv(0), ZMQ.CHARSET).trim();
                if ((task_nbr / 10) * 10 == task_nbr) {
                    System.out.print(":");
                }
                else {
                    System.out.print(".");
                }
            }

            //  Calculate and report duration of batch
            long tend = System.currentTimeMillis();

            System.out.println(
                "\nTotal elapsed time: " + (tend - tstart) + " msec"
            );
        }
    }
}
```

一个批次的平均成本是 5 秒。当我们启动 1、2 或 4 个workers 时，我们会从接收器中得到如下结果：

+ 1 个worker：总耗时：5034 毫秒。
+ 2 个worker：总耗用时间：2421 毫秒。
+ 4 个worker：总耗时：1018 毫秒。

让我们更详细地看一下这段代码的一些方面：

+ workers在上游连接到ventilator，在下游连接到sink。这意味着您可以任意添加workers。但如果workers绑定到他们的终端，为了添加新的workers您将需要 1.更多终端和 2.每次添加workers时修改ventilator和sink。我们说ventilator和sink应该是是我们架构的*稳定部分*，而workers是它的*动态部分*。
+ 我们必须将批处理的开始与所有正在运行的工作线程同步。这是 ZeroMQ 中相当常见的问题，没有简单的解决方案。 zmq_connect 方法需要一定的时间。因此，当一组workers连接到ventilator时，第一个成功连接的worker将在短时间内收到大量消息，而其他worker还在连接。如果您不以某种方式同步批处理的开始，则系统根本不会并行运行。尝试取消ventilator中的等待，看看会发生什么。
+ ventilator的 PUSH 套接字将任务均匀地分配给workers（假设他们在批次开始出去*之前*都已连接）。这称为*负载均衡*，我们将再次详细讨论它。

接收器的 PULL 套接字均匀地收集workers的结果。这称为*公平队列*。
Figure 6 - Fair Queuing
{% asset_img fig6.png Figure 6 %}

pipeline模式还表现出“slow joiner”综合症，导致人们指责 PUSH 套接字没有正确负载均衡。如果您使用 PUSH 和 PULL，并且您的一个 worker 收到的消息比其他的多，那是因为该 PULL socket 加入的速度比其他的快，并且在其他人设法连接之前抓取了很多消息。如果您想要适当的负载均衡，您可能需要查看Chapter 3 - Advanced Request-Reply Patterns.

## 使用 ZeroMQ 编程

&emsp;&emsp;看过一些示例后，您一定渴望开始在某些应用程序中使用 ZeroMQ。在你开始之前，深呼吸，冷静一下，并思考一些基本的建议，这些建议可以为你节省很多压力和困惑。

+ 逐步学习 ZeroMQ。它只是一个简单的 API，但它隐藏了一个充满可能性的世界。慢慢把握各种可能性并掌握每一种可能性。
+ 写出漂亮的代码。丑陋的代码隐藏了问题，让其他人很难帮助你。您可能会习惯无意义的变量名，但阅读您代码的人不会。使用真实的名称，而不是说“我太粗心了，无法告诉你这个变量的真正用途”。使用一致的缩进和干净的布局。写出漂亮的代码，你的世界会更舒服。
+ 边做边测试。当你的程序不工作时，你应该知道这五行应该归咎于什么。当您使用 ZeroMQ 时尤其如此，您开始的几次尝试通常会遇到麻烦。
+ 当你发现事情不像预期的那样工作时，把你的代码分成几段，测试每一段，看看哪个不工作。 ZeroMQ 使您可以编写本质上模块化的代码；充分利用这一点。
+ 根据需要进行抽象（类、方法等）。如果您复制/粘贴大量代码，您也会复制/粘贴错误。

### 获得正确的上下文

&emsp;&emsp;ZeroMQ 应用程序总是首先创建一个*上下文*，然后使用它来创建套接字。在 C 中，它是 [zmq_ctx_new()](http://api.zeromq.org/master:zmq-ctx-new) 调用。您应该在您的流程中创建和使用一个上下文。从技术上讲，上下文是单个进程中所有套接字的容器，并充当 **inproc** 套接字的传输，这是在一个进程中连接线程的最快方式。如果在运行时一个进程有两个上下文，它们就像单独的 ZeroMQ 实例。如果这正是您想要的，好的，但请记住：

**在进程开始前调用 [zmq_ctx_new()](http://api.zeromq.org/master:zmq-ctx-new) 一次, 并且在结束时调用 [zmq_ctx_destroy()](http://api.zeromq.org/master:zmq-ctx-destroy) 一次。**

如果您正在使用 fork() 系统调用，请在 fork() *之后*和子进程代码的开头执行 [zmq_ctx_new()](http://api.zeromq.org/master:zmq-ctx-new)。一般来说，你想在子进程中做有趣（ZeroMQ）的事情，然后在父进程中做无聊的进程管理。

### 彻底退出

&emsp;&emsp;优秀的程序员与优秀的杀手有着相同的座右铭：完成工作后总是清理。当你在 Python 之类的语言中使用 ZeroMQ 时，东西会自动为你释放。但是在使用 C 时，您必须在完成后小心地释放对象，否则会出现内存泄漏、应用程序不稳定以及通常的不良业障。

&emsp;&emsp;内存泄漏是一回事，但 ZeroMQ 对退出应用程序的方式非常挑剔。原因是技术性的和痛苦的，但结果是，如果您让任何套接字保持打开状态，[zmq_ctx_destroy()](http://api.zeromq.org/master:zmq-ctx-destroy) 函数将永远挂起。即使您关闭所有套接字，[zmq_ctx_destroy()](http://api.zeromq.org/master:zmq-ctx-destroy) 默认情况下也会在有挂起的连接或发送时永远等待，除非您在关闭它们之前将这些套接字上的 LINGER 设置为0

&emsp;&emsp;我们需要担心的 ZeroMQ 对象是消息、套接字和上下文。幸运的是，它非常简单，至少在简单的程序中是这样：

+ 尽可能使用 [zmq_send()](http://api.zeromq.org/master:zmq-send) 和 [zmq_recv()](http://api.zeromq.org/master:zmq-recv)，因为它避免了使用 zmq_msg_t 对象的需要。
+ 如果您确实使用了 [zmq_msg_recv()](http://api.zeromq.org/master:zmq-msg-recv)，请在使用完后立即通过调用 [zmq_msg_close()](http://api.zeromq.org/master:zmq-msg-close) 释放接收到的消息。
+ 如果您要打开和关闭大量套接字，这可能表明您需要重新设计应用程序。在某些情况下，除非您销毁上下文，否则不会释放套接字句柄。
+ 退出程序时，关闭套接字，然后调用 [zmq_ctx_destroy()](http://api.zeromq.org/master:zmq-ctx-destroy)。这破坏了上下文。

至少对于 C 开发来说是这样。在具有自动对象销毁的语言中，套接字和上下文将在您离开作用域时被销毁。如果您使用异常，则必须在*finally*之类的东西中进行清理，与任何资源相同。

如果你在做多线程工作，它会变得比这更复杂。我们将在下一章中讨论多线程，但是你们中的一些人会无视警告，没学会爬就开始跑，下面是在多线程 ZeroMQ 应用程序中干净退出的方法，虽然他出自一份快速却肮脏的指南。

首先，不要尝试从多个线程使用同一个套接字。请不要解释为什么您认为这会非常有趣，请不要这样做。接下来，您需要关闭每个有正在进行请求的套接字。正确的方法是设置一个较低的 LINGER 值（1 秒），然后关闭套接字。如果您的编程语言绑定在您销毁上下文时没有自动为您执行此操作，我建议您发送补丁。

最后，销毁上下文。这将导致附加线程（即共享相同上下文）中的任何阻塞接收或轮询或发送以错误返回。捕获该错误，然后设置 linger 并关闭该线程中的套接字，然后退出。不要两次破坏相同的上下文。主线程中的 zmq_ctx_destroy 将阻塞，直到它知道的所有套接字都安全关闭。

瞧！这足够复杂和痛苦，以至于任何称职的编程语言绑定创作者都会自动执行此操作，并使套接字关闭操作变得不必要。

## 为什么我们需要ZeroMQ

&emsp;&emsp;现在你已经看到了 ZeroMQ 是如何运行的，让我们回到“为什么”。如今，许多应用程序由跨越某种网络（LAN 或 Internet）的组件组成。 如此多的应用程序开发人员最终会执行某种消息传递。 一些开发人员使用消息队列产品，但大多数时候他们自己做，使用 TCP 或 UDP。 这些协议并不难使用，但是从 A 到 B 发送几个字节与以任何一种可靠的方式进行消息传递之间存在很大差异。

&emsp;&emsp;让我们看看当我们开始使用原始 TCP 连接片段时我们面临的典型问题。 任何可重用的消息传递层都需要解决所有或其中大部分问题：

+ 我们如何处理 I/O？我们的应用程序是阻塞还是在后台处理 I/O？这是一个关键的设计决策。阻塞 I/O 会创建不能很好扩展的架构。但是后台 I/O 很难做到正确。
+ 我们如何处理动态组件，即暂时消失的部分？我们是否正式将组件拆分为“客户端”和“服务端”，并要求服务端不能消失？那么如果我们想将服务端连接到服务端呢？我们是否每隔几秒钟就尝试重新连接？
+ 我们如何在线路上表示消息？我们如何构建数据以使其易于写入和读取，避免缓冲区溢出，对小消息或许有效，但对于戴着派对帽的跳舞猫的很大的视频也足够了吗？
+ 我们如何处理无法立即传递的消息？特别是，如果我们正在等待一个组件重新上线的时候？我们是丢弃消息，将它们放入数据库，还是放入内存队列？
+ 我们在哪里存储消息队列？如果从队列中读取的组件非常慢并导致我们的队列堆积，会发生什么情况？那我们的策略是什么？
+ 我们如何处理丢失的消息？我们是等待新数据，请求重新发送，还是构建某种可靠性层来确保消息不会丢失？如果该层本身崩溃怎么办？
+ 如果我们需要使用不同的网络传输怎么办。比如说，多播而不是 TCP 单播？还是IPv6？我们是否需要重写应用程序，或者传输是在某个层中抽象出来的？
+ 我们如何路由消息？我们可以向多个对等点发送相同的消息吗？我们可以将回复发送回原始请求者吗？
+ 我们如何为另一种语言编写 API？我们是重新实现一个wire protocol(因为代码层面的数据（链表、队列、二叉树）都是结构化的，但网络层看到的都是二进制流，所以把结构化的数据序列化为二进制流发送出去，并且对方也能以同样的格式反序列化出来，这就是wire protocol)还是重新打包一个库？如果是前者，如何保证栈高效稳定？如果是后者，我们如何保证互操作性？
+ 我们如何表示数据以便它可以在不同架构之间读取？我们是否对数据类型强制执行特定编码？那么这个消息传递系统网络中的更高层还有多大区别？
+ 我们如何处理网络错误？我们是等待并重试，还是默默地忽略它们，还是中止？

&emsp;&emsp;举一个典型的例子，像Hadoop Zookeeper这样的开源项目，阅读src/c/src/zookeeper.c中的C API代码。当我在 2013 年 1 月阅读这段代码时，它有 4,200 行，其中包含一个未记录的客户端/服务端网络通信协议。我认为它很高效，因为它使用 poll 而不是 select。但实际上，Zookeeper 应该使用通用消息传递层和明确记录的线路级协议。团队一遍又一遍地构建这个特殊的重复轮子是非常浪费的。

&emsp;&emsp;但是如何制作可重用的消息传递层呢？为什么，当如此多的项目需要这项技术时，人们仍然通过在他们的代码中驱动 TCP 套接字并一遍又一遍地解决那长长的列表中的问题来艰难地做到这一点？

&emsp;&emsp;事实证明，构建可重用的消息队列系统真的很困难，这就是为什么很少有 FOSS 项目尝试过，以及为什么商业消息队列产品复杂、昂贵、不灵活和脆弱的原因。 2006 年，iMatix 设计了 ​​[AMQP](https://www.amqp.org/)，它开始为 FOSS 开发人员提供可能是消息系统的第一个可重用配方。 AMQP 比许多其他设计工作得更好，[但仍然相对复杂、昂贵和脆弱](https://web.archive.org/web/20190620095529/www.imatix.com/articles:whats-wrong-with-amqp)。学习使用需要数周时间，而创建稳定的、即便任务变得棘手也不会崩溃的架构，需要数月的时间。

Figure 7 - Messaging as it Starts
{% asset_img fig7.png Figure 7 %}

&emsp;&emsp;大多数消息队列项目，如 AMQP，试图以可重用的方式解决这一长串问题，这是通过发明一个新概念“broker”来实现的，该概念可以进行寻址、路由和排队。这导致在一些未记录的协议之上的客户端/服务端协议或一组 API 允许应用程序与此broker对话。broker是降低大型网络复杂性的好东西。但是，向 Zookeeper 等产品添加基于broker的消息队列会使情况变得更糟，而不是更好。这将意味着添加一个额外的大组件和一个新的单点故障。broker迅速成为瓶颈和需要管理的新风险。如果软件支持，我们可以添加第二个、第三个和第四个broker并制定一些故障转移方案。人们会这样做。它创造了更多动人的碎片和更多的复杂性。

&emsp;&emsp;以broker为中心的设置需要自己的运营团队。您确实需要日夜观察broker，并在他们开始行为不端时用棍子殴打他们。你需要组件，你需要备用组件，你需要有人来管理这些组件。仅对于由多个团队花费数年时间构建的具有许多可拆分模组的大型应用程序才值得这样做。

Figure 8 - Messaging as it Becomes
{% asset_img fig8.png Figure 8 %}

因此，中小型应用程序开发人员便陷入了困境。他们要么避免网络编程，要么制作不可扩展的单体应用程序。或者他们跳入网络编程并制作难以维护的脆弱、复杂的应用程序。或者他们押注消息队列产品，最终得到依赖于昂贵且容易损坏的技术的可扩展应用程序。一直没有真正好的选择，这也许就是为什么消息在很大程度上停留在上个世纪并激起强烈的情绪：用户的负面情绪，销售支持和许可证的人的喜悦。

我们需要的是一种能够完成消息队列工作的东西，但它以一种简单而廉价的方式来完成，它可以在任何应用程序中工作，成本接近于零。它应该是一个您只需链接库，没有任何其他依赖项。没有额外的移动件，所以没有额外的风险。它应该在任何操作系统上运行并使用任何编程语言。

这就是 ZeroMQ：一个高效的、可嵌入的库，它解决了应用程序需要在整个网络中变得具有良好伸缩性的大部分问题，而无需太多成本。

具体来说：

+ 它在后台线程中异步处理 I/O。它们使用无锁数据结构与应用程序线程通信，因此并发 ZeroMQ 应用程序不需要锁、信号量或其他等待状态。
+ 组件可以动态地来来去去，ZeroMQ 将自动重新连接。这意味着您可以按任何顺序启动组件。您可以创建“面向服务的架构”(SOA)，服务可以随时加入和离开网络。
+ 它在需要时自动将消息排队。它可以智能地做到这一点，在排队之前将消息尽可能靠近接收者。
+ 它有处理超满队列（称为“high water mark”）的方法。当队列已满时，ZeroMQ 会自动阻止发送者或丢弃消息，具体取决于您正在执行的消息传递类型（所谓的“模式”）。
+ 它让您的应用程序可以通过任意传输方式相互通信：TCP、多播、进程内、进程间。您无需更改代码即可使用不同的传输。
+ 它使用取决于消息传递模式的不同策略安全地处理慢速/阻塞的读取器。
+ 它允许您使用各种模式（例如请求-回复和发布-订阅）来路由消息。这些模式是您创建拓扑和网络结构的方式。
+ 它使您可以创建代理以通过一次调用来排队、转发或捕获消息。代理可以降低网络互连的复杂性。
+ 它完全按照发送的方式传递整个消息，在线路上使用简单的框架。如果你写一条 10k 的消息，你会收到一条 10k 的消息。
+ 它不会对消息强加任何格式。它们是从零到千兆字节的 blob。当您想表示数据时，您可以选择一些其他产品，例如 msgpack、Google 的协议缓冲区等。
+ 它通过在有意义的情况下自动重试来智能地处理网络错误。
+ 它可以减少您的碳足迹。用更少的 CPU 做更多的事情意味着你的硬件使用更少的能量，你可以让你的旧机器使用更长时间。 Al Gore 会喜欢 ZeroMQ。

实际上，ZeroMQ 所做的远不止这些。它对您如何开发具有网络功能的应用程序具有颠覆性的影响。从表面上看，它是一个受套接字启发的 API，您可以在其上执行 [zmq_recv()](http://api.zeromq.org/master:zmq-recv) 和 [zmq_send()](http://api.zeromq.org/master:zmq-send)。消息处理会迅速成为中心循环，您的应用程序很快分解为一组消息处理任务。它优雅而自然。它可以扩展：这些任务中的每一个都映射到一个节点，节点之间通过任意传输相互通信。一个进程中的两个节点（节点是一个线程），一个机器上的两个节点（节点是一个进程），或者一个网络上的两个节点（节点是一台机器）——都是一样的，应用程序代码没有变化。

## 套接字可扩展性

让我们看看 ZeroMQ 的可扩展性。 这是一个 shell 脚本，它启动天气服务器，然后并行启动一堆客户端：

```shell
wuserver &
wuclient 12345 &
wuclient 23456 &
wuclient 34567 &
wuclient 45678 &
wuclient 56789 &
```

在客户端运行时，我们使用 top 命令查看活动进程，我们会看到类似（在 4 核机器上）的内容：

```shell
PID  USER  PR  NI  VIRT  RES  SHR S %CPU %MEM   TIME+  COMMAND
7136  ph   20   0 1040m 959m 1156 R  157 12.0 16:25.47 wuserver
7966  ph   20   0 98608 1804 1372 S   33  0.0  0:03.94 wuclient
7963  ph   20   0 33116 1748 1372 S   14  0.0  0:00.76 wuclient
7965  ph   20   0 33116 1784 1372 S    6  0.0  0:00.47 wuclient
7964  ph   20   0 33116 1788 1372 S    5  0.0  0:00.25 wuclient
7967  ph   20   0 33072 1740 1372 S    5  0.0  0:00.35 wuclient
```

让我们考虑一下这里发生的事情。 天气服务器只有一个套接字，但在这里我们让它并行地向五个客户端发送数据。 我们可以有数千个并发客户端。 服务端应用程序看不到它们，不直接与它们交谈。 所以 ZeroMQ 套接字就像一个小服务器，默默地接受客户端请求，并以网络可以处理的速度将数据推送给他们。 它是一个多线程服务器，更多地压榨您CPU的性能。

## 从 ZeroMQ v2.2 升级到 ZeroMQ v3.2

### 兼容的变化

这些更改不会直接影响现有的应用程序代码：

发布 - 订阅过滤现在在发布者端而不是订阅者端完成。 这在许多 pub-sub 用例中显着提高了性能。 您可以安全地混合使用 v3.2 和 v2.1/v2.2 发布者和订阅者。

ZeroMQ v3.2 有许多新的 API 方法（[zmq_disconnect()](http://api.zeromq.org/master:zmq-disconnect)、[zmq_unbind()](http://api.zeromq.org/master:zmq-unbind)、[zmq_monitor()](http://api.zeromq.org/master:zmq-monitor)、[zmq_ctx_set()](http://api.zeromq.org/master:zmq-ctx-set) 等）

### 不兼容的变化

这些是影响应用程序和语言绑定的主要领域：

+ 更改的发送/接收方法：[zmq_send()](http://api.zeromq.org/master:zmq-send) 和 [zmq_recv()](http://api.zeromq.org/master:zmq-recv) 具有不同的、更简单的接口，旧功能现在由 [zmq_msg_send()](http://api.zeromq.org/master:zmq-msg-send) 和 [zmq_msg_recv()](http://api.zeromq.org/master:zmq-msg-recv) 提供。故障现象：编译错误。解决方案：修复您的代码。
+ 这两种方法在成功时返回正值，在错误时返回 -1。在 v2.x 中，它们总是在成功时返回零。症状：实际工作正常时出现明显错误。解决方案：严格测试返回码 = -1，而不是非零。
+ [zmq_poll()](http://api.zeromq.org/master:zmq-poll) 现在等待以毫秒为单位，而不是微秒。症状：应用程序停止响应（实际上响应慢了 1000 倍）。解决方案：在所有 zmq_poll 调用中使用下面定义的 ZMQ_POLL_MSEC 宏。
+ ZMQ_NOBLOCK 现在被称为 ZMQ_DONTWAIT。症状：在 ZMQ_NOBLOCK 宏上编译失败。
+ ZMQ_HWM 套接字选项现在分为 ZMQ_SNDHWM 和 ZMQ_RCVHWM。症状：在 ZMQ_HWM 宏上编译失败。
+ 大多数但不是所有 [zmq_getsockopt()](http://api.zeromq.org/master:zmq-getsockopt) 选项现在都是整数值。症状：在 zmq_setsockopt 和 zmq_getsockopt 上返回运行时错误。
+ ZMQ_SWAP 选项已被删除。症状：在 ZMQ_SWAP 上编译失败。解决方案：重新设计任何使用此功能的代码。

### 推荐的Shim 宏

对于希望同时在 v2.x 和 v3.2 上运行的应用程序，例如language bindings，我们的建议是尽可能模拟 v3.2。 以下是 C 宏定义，可帮助您的 C/C++ 代码跨两个版本工作（取自 [CZMQ](http://czmq.zeromq.org/)）：

```shell
#ifndef ZMQ_DONTWAIT
#   define ZMQ_DONTWAIT     ZMQ_NOBLOCK
#endif
#if ZMQ_VERSION_MAJOR == 2
#   define zmq_msg_send(msg,sock,opt) zmq_send (sock, msg, opt)
#   define zmq_msg_recv(msg,sock,opt) zmq_recv (sock, msg, opt)
#   define zmq_ctx_destroy(context) zmq_term(context)
#   define ZMQ_POLL_MSEC    1000        //  zmq_poll is usec
#   define ZMQ_SNDHWM ZMQ_HWM
#   define ZMQ_RCVHWM ZMQ_HWM
#elif ZMQ_VERSION_MAJOR == 3
#   define ZMQ_POLL_MSEC    1           //  zmq_poll is msec
#endif
```

## 警告：不稳定的范式！

传统的网络编程建立在一个通用假设之上，即一个套接字与一个连接、一个对等方对话。有组播协议，但这些都是诡异的。当我们假设“一个套接字=一个连接”时，我们会以某种方式扩展我们的架构。我们创建逻辑线程，其中每个线程与一个套接字、一个对等体一起工作。我们将智能化的自动逻辑和状态放在这些线程中。

在 ZeroMQ 世界中，套接字是快速小型后台通信引擎的入口，这些引擎自动为您管理一整套连接。您无法查看、使用、打开、关闭这些连接或将状态附加到这些连接。无论您使用阻塞发送或接收，还是轮询，您只能与套接字对话，而不是它为您管理的连接。连接是私有且不可见的，这是 ZeroMQ 可扩展性的关键。

这是因为您的代码与套接字通信，然后可以处理任何网络协议上的任意数量的连接，而无需更改。 ZeroMQ 中的消息传递模式比应用程序代码中的消息传递模式代价更低。

所以一般假设不再适用。当您阅读代码示例时，您的大脑会尝试将它们映射到您所知道的内容。你会读到“socket”并想“啊，这代表了到另一个节点的连接”。那是错的。你会读到“线程”，你的大脑会再次想，“啊，一个线程代表与另一个节点的连接”，你的大脑又会出错。

如果您是第一次阅读本指南，请意识到在您实际编写一两天（也许三四天）ZeroMQ 代码之前，您可能会感到困惑，尤其是 ZeroMQ 为您制作的东西是多么简单，并且您可能会尝试将一般假设强加于 ZeroMQ，但它不会起作用。然后你会体验到你的思想上启蒙和对ZeroMQ信任建立的时刻，当一切都变得清晰时，那个 zap-pow-kaboom satori 范式转变时刻。
