---


---

<blockquote>
<p><a href="https://kangyupl.gitee.io/cs144.github.io/assignments/lab2.pdf">https://kangyupl.gitee.io/cs144.github.io/assignments/lab2.pdf</a></p>
</blockquote>
<h1 id="overview">1 overview</h1>
<p>Now, in Lab 2, you’ll implement the part of TCP that handles the inbound byte-stream: the TCPReceiver. You’ve already done most of the “algorithmic” work involved in this when you wrote the StreamReassembler and the ByteStream; this week is mostly about wiring those classes up to the format of TCP.</p>
<p>This will involve thinking about <strong>how TCP will represent each byte’s place in the stream—known as a “sequence number”.</strong> The TCPReceiver is responsible for telling the sender<br>
(a) how much of the inbound byte stream it’s been able to assemble successfully (this is called “acknowledgment”) and<br>
(b) what range of bytes the sender is allowed to send right now (the “flow control window”).</p>
<h1 id="the-tcp-receiver">3  The TCP Receiver</h1>
<blockquote>
<p>TCP is a protocol that reliably conveys a pair of flow-controlled byte streams (one in each direction) over unreliable datagrams. Two parties participate in the TCP connection, and each party acts as both “sender” (of its own outgoing byte-stream) and “receiver” (of an incoming byte-stream) at the same time. The two parties are called the “endpoints” of the connection, or the “peers.”</p>
</blockquote>
<blockquote>
<p>lab2要实现的TCP receiver模块用于接收端处理收到的TCP数据段。该模块负责提取数据段中的信息，并使用StreamReassembler模块将数据写入ByteStream中。<br>
<a href="https://zhuanlan.zhihu.com/p/265156728">CS144计算机网络lab2：TCP receiver</a></p>
</blockquote>
<p>讲义解析：</p>
<h1 id="reference">reference</h1>
<ol>
<li><a href="https://www.cnblogs.com/kangyupl/p/stanford_cs144_labs.html#2825868232">LAB2#</a></li>
<li><a href="https://zhuanlan.zhihu.com/p/265156728">CS144计算机网络lab2：TCP receiver</a></li>
</ol>

