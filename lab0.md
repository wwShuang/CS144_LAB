---


---

<blockquote>
<p><a href="https://kangyupl.gitee.io/cs144.github.io/assignments/lab0.pdf">https://kangyupl.gitee.io/cs144.github.io/assignments/lab0.pdf</a><br>
一些过程略</p>
</blockquote>
<h2 id="fetch-a-web-page">2.1 Fetch a Web page</h2>
<p><strong>这里是http request 的步骤</strong></p>
<ol>
<li><code>telnet cs144.keithw.org http</code>. This tells the telnet program to open a reliable byte stream between your computer and another computer (named <a href="http://cs144.keithw.org">cs144.keithw.org</a>), and with a particular service running on that computer: the “http” service, for the Hyper-Text Transfer Protocol, used by the World Wide Web.</li>
<li><code>GET /hello HTTP/1.1</code> the path part</li>
<li><code>Host: cs144.keithw.org</code> the host part</li>
<li>Hit the Enter key one more time: . This tells the server that you are done with your HTTP request.</li>
</ol>
<h2 id="send-yourself-an-email">2.2 Send yourself an email</h2>
<p>jump</p>
<h1 id="listening-and-connecting">2.3 Listening and connecting</h1>
<ol>
<li>In one terminal window, run  <code>netcat -v -l -p 9090</code> on your VM.</li>
<li>Leave netcat running. In another terminal window, run <code>telnet localhost 9090</code> (also on your VM).</li>
</ol>
<h1 id="writing-a-network-program-using-an-os-stream-socket">3 Writing a network program using an OS stream socket</h1>
<blockquote>
<p><a href="https://zhuanlan.zhihu.com/p/37633767">https://zhuanlan.zhihu.com/p/37633767</a></p>
</blockquote>
<p>write a short program that fetches a Web page over the Internet: create a reliable bidirectional in-order byte stream between two programs, one running on your computer, and the other on a different computer across the Internet.<br>
This feature is known as a stream socket.</p>
<p>cmake的意义：</p>
<h2 id="writing-webget">3.4 Writing webget</h2>
<blockquote>
<p><a href="https://zhuanlan.zhihu.com/p/257283830">CS144计算机网络lab0：networking热身</a><br>
<a href="https://www.cnblogs.com/kangyupl/p/stanford_cs144_labs.html">【计算机网络】Stanford CS144 Lab Assignments 学习笔记</a><br>
<a href="https://cs144.github.io/doc/lab0/class_t_c_p_socket.html">TCPSocket的示例代码</a></p>
</blockquote>
<blockquote>
<p>分析：让我自己写是肯定写不出来的，先分析一遍答案，再写<br>
要是说的话 就是和2.1 是一样的</p>
</blockquote>
<pre class=" language-c"><code class="prism ++ language-c"><span class="token keyword">void</span> <span class="token function">get_URL</span><span class="token punctuation">(</span><span class="token keyword">const</span> string <span class="token operator">&amp;</span>host<span class="token punctuation">,</span> <span class="token keyword">const</span> string <span class="token operator">&amp;</span>path<span class="token punctuation">)</span> <span class="token punctuation">{</span>

    TCPSocket sock<span class="token punctuation">{</span><span class="token punctuation">}</span><span class="token punctuation">;</span>
    sock<span class="token punctuation">.</span><span class="token function">connect</span><span class="token punctuation">(</span><span class="token function">Address</span><span class="token punctuation">(</span>host<span class="token punctuation">,</span><span class="token string">"http"</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  <span class="token comment">//host</span>
    sock<span class="token punctuation">.</span><span class="token function">write</span><span class="token punctuation">(</span><span class="token string">"GET "</span><span class="token operator">+</span>path<span class="token operator">+</span><span class="token string">" HTTP/1.1\r\nHost: "</span><span class="token operator">+</span>host<span class="token operator">+</span><span class="token string">"\r\n\r\n"</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">// the path+ hit one more time</span>
    sock<span class="token punctuation">.</span><span class="token function">shutdown</span><span class="token punctuation">(</span>SHUT_WR<span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token keyword">while</span><span class="token punctuation">(</span><span class="token operator">!</span>sock<span class="token punctuation">.</span><span class="token function">eof</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">{</span>
        cout<span class="token operator">&lt;&lt;</span>sock<span class="token punctuation">.</span><span class="token function">read</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
    sock<span class="token punctuation">.</span><span class="token function">close</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token keyword">return</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
<p>attention:<br>
注意在客户端socket发送完请求报文之后，要使用shutdown方法破坏客户端和服务器套接字之间的连接；如果仅用close方法，则只是关闭socket连接的某一个标识，连接本身还是开着的（我不太确定Connection: close\r\n\r\n和后面的socket.shutdown作用是否重复，不深究）。<br>
套接字本身相当于一个文件描述符，程序通过它对某个“文件”（字节流抽象）进行写入和读出的操作。我们通过TCPSocket类中的eof方法判断套接字中写入的数据是否已经到达了EOF，只要还没有，就仍要使用read方法进行读取（在下一个小实验中，我们会更加清楚为什么要这么做）。</p>
<h1 id="an-in-memory-reliable-byte-stream">4 An in-memory reliable byte stream</h1>
<blockquote>
<p><a href="https://hexyoungs.club/blog/cs144-lab0/">CS144 Lab0 学习笔记</a><br>
<a href="https://zhuanlan.zhihu.com/p/257283830">CS144计算机网络lab0：networking热身</a><br>
<a href="https://zhuanlan.zhihu.com/p/77981148">C++中queue和deque的区别</a><br>
<a href="https://en.cppreference.com/w/cpp/types/size_t">size_t</a><br>
<a href="https://link.zhihu.com/?target=https%3A//jeremybai.github.io/blog/2014/09/10/size-t">为什么size_t重要？（Why size_t matters）</a><br>
<a href="http://www.cplusplus.com/reference/string/string/assign/">string 语法</a><br>
<a href="http://www.cplusplus.com/reference/deque/deque/">deque语法</a></p>
</blockquote>
<p>第二个任务要求实现一个有序字节流类（in-order byte stream），使之支持读写、容量控制。由于是单线程，并且不考虑并发读写，我们只需要利用 cpp 标准库里面的容器(这里用了 dequeue )保存数据，然后适配所有的接口并且通过测试用例即可。<br>
这个字节流类似于一个带容量的队列，从一头读，从另一头写。当流中的数据达到容量上限时，便无法再写入新的数据。特别的，写操作被分为了peek和pop两步。<br>
peek为从头部开始读取指定数量的字节，pop为弹出指定数量的字节。</p>
<p>（看着这块自己写代码，不要看答案）逻辑：</p>
<ol>
<li>写操作，如果数据大于剩余空闲空间大小，则只等于最大剩余容量</li>
<li>记录数量</li>
<li>向 buffer  里写入数据</li>
</ol>
<pre class=" language-ruby"><code class="prism  language-ruby"><span class="token operator">+</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">-</span><span class="token operator">+</span>  <span class="token operator">&lt;</span><span class="token operator">-</span> capacity
<span class="token operator">|</span>                   <span class="token operator">|</span>
<span class="token operator">|</span>                   <span class="token operator">|</span>
<span class="token operator">|</span>                   <span class="token operator">|</span>
<span class="token operator">+</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">-</span><span class="token operator">+</span>  <span class="token operator">&lt;</span><span class="token operator">-</span> buffer<span class="token punctuation">.</span><span class="token function">size</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
<span class="token operator">|</span>					<span class="token operator">|</span>
<span class="token operator">+</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">-</span><span class="token operator">+</span>  <span class="token operator">&lt;</span><span class="token operator">-</span> <span class="token number">0</span>
</code></pre>
<pre class=" language-c"><code class="prism ++ language-c"><span class="token comment">// .hh 命名参数 </span>
class ByteStream <span class="token punctuation">{</span>
  private<span class="token punctuation">:</span>
    <span class="token comment">// Your code here -- add private members as necessary.</span>
    std<span class="token punctuation">:</span><span class="token punctuation">:</span>deque<span class="token operator">&lt;</span><span class="token keyword">char</span><span class="token operator">&gt;</span> _buffer <span class="token operator">=</span> <span class="token punctuation">{</span><span class="token punctuation">}</span><span class="token punctuation">;</span>
    size_t _capacity <span class="token operator">=</span> <span class="token number">0</span><span class="token punctuation">;</span>
    size_t _read_count <span class="token operator">=</span> <span class="token number">0</span><span class="token punctuation">;</span>
    size_t _write_count <span class="token operator">=</span> <span class="token number">0</span><span class="token punctuation">;</span>
    bool _input_ended_flag <span class="token operator">=</span> false<span class="token punctuation">;</span>
    bool _error <span class="token operator">=</span> false<span class="token punctuation">;</span>  <span class="token comment">//!&lt; Flag indicating that the stream suffered an error.</span>
    <span class="token comment">//......</span>

<span class="token comment">// .cc 文件 程序</span>
ByteStream<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">ByteStream</span><span class="token punctuation">(</span><span class="token keyword">const</span> size_t capacity<span class="token punctuation">)</span> <span class="token punctuation">:</span> <span class="token function">_capacity</span><span class="token punctuation">(</span>capacity<span class="token punctuation">)</span> <span class="token punctuation">{</span><span class="token punctuation">}</span> <span class="token comment">// 这里为什么？</span>

size_t ByteStream<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">write</span><span class="token punctuation">(</span><span class="token keyword">const</span> string <span class="token operator">&amp;</span>data<span class="token punctuation">)</span> <span class="token punctuation">{</span>
    size_t len <span class="token operator">=</span> data<span class="token punctuation">.</span><span class="token function">length</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">// 命名是加上变量类型</span>
    <span class="token comment">// 这块记不住加不加括号，现默认都加上</span>
    <span class="token keyword">if</span> <span class="token punctuation">(</span>len <span class="token operator">&gt;</span> _capacity <span class="token operator">-</span> _buffer<span class="token punctuation">.</span><span class="token function">size</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        len <span class="token operator">=</span> _capacity <span class="token operator">-</span> _buffer<span class="token punctuation">.</span><span class="token function">size</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
    _write_count <span class="token operator">+</span><span class="token operator">=</span> len<span class="token punctuation">;</span>
    <span class="token keyword">for</span> <span class="token punctuation">(</span>size_t i <span class="token operator">=</span> <span class="token number">0</span><span class="token punctuation">;</span> i <span class="token operator">&lt;</span> len<span class="token punctuation">;</span> i<span class="token operator">++</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        _buffer<span class="token punctuation">.</span><span class="token function">push_back</span><span class="token punctuation">(</span>data<span class="token punctuation">[</span>i<span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
    <span class="token keyword">return</span> len<span class="token punctuation">;</span>
<span class="token punctuation">}</span>

<span class="token comment">//! \param[in] len bytes will be copied from the output side of the buffer</span>
string ByteStream<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">peek_output</span><span class="token punctuation">(</span><span class="token keyword">const</span> size_t len<span class="token punctuation">)</span> <span class="token keyword">const</span> <span class="token punctuation">{</span>
    size_t length <span class="token operator">=</span> len<span class="token punctuation">;</span> <span class="token comment">// 他这里用了变量获取常量值，方便下面操作</span>
    <span class="token keyword">if</span> <span class="token punctuation">(</span>length <span class="token operator">&gt;</span> _buffer<span class="token punctuation">.</span><span class="token function">size</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        length <span class="token operator">=</span> _buffer<span class="token punctuation">.</span><span class="token function">size</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
    <span class="token keyword">return</span> <span class="token function">string</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">.</span><span class="token function">assign</span><span class="token punctuation">(</span>_buffer<span class="token punctuation">.</span><span class="token function">begin</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">,</span> _buffer<span class="token punctuation">.</span><span class="token function">begin</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token operator">+</span> length<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>

<span class="token comment">//! \param[in] len bytes will be removed from the output side of the buffer</span>
<span class="token keyword">void</span> ByteStream<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">pop_output</span><span class="token punctuation">(</span><span class="token keyword">const</span> size_t len<span class="token punctuation">)</span> <span class="token punctuation">{</span>
    size_t length <span class="token operator">=</span> len<span class="token punctuation">;</span>
    <span class="token keyword">if</span> <span class="token punctuation">(</span>length <span class="token operator">&gt;</span> _buffer<span class="token punctuation">.</span><span class="token function">size</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        length <span class="token operator">=</span> _buffer<span class="token punctuation">.</span><span class="token function">size</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
    _read_count <span class="token operator">+</span><span class="token operator">=</span> length<span class="token punctuation">;</span>  
    <span class="token comment">// 这里涉及到了 _read_count,读取数量相当于出战的数量</span>
    <span class="token keyword">while</span> <span class="token punctuation">(</span>length<span class="token operator">--</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
        _buffer<span class="token punctuation">.</span><span class="token function">pop_front</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
    <span class="token keyword">return</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>

<span class="token keyword">void</span> ByteStream<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">end_input</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span> _input_ended_flag <span class="token operator">=</span> true<span class="token punctuation">;</span> <span class="token punctuation">}</span>

bool ByteStream<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">input_ended</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token keyword">const</span> <span class="token punctuation">{</span> <span class="token keyword">return</span> _input_ended_flag<span class="token punctuation">;</span> <span class="token punctuation">}</span>

size_t ByteStream<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">buffer_size</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token keyword">const</span> <span class="token punctuation">{</span> <span class="token keyword">return</span> _buffer<span class="token punctuation">.</span><span class="token function">size</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token punctuation">}</span>

bool ByteStream<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">buffer_empty</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token keyword">const</span> <span class="token punctuation">{</span> <span class="token keyword">return</span> _buffer<span class="token punctuation">.</span><span class="token function">size</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token operator">==</span> <span class="token number">0</span><span class="token punctuation">;</span> <span class="token punctuation">}</span>

bool ByteStream<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">eof</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token keyword">const</span> <span class="token punctuation">{</span> <span class="token keyword">return</span> <span class="token function">buffer_empty</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token operator">&amp;&amp;</span> <span class="token function">input_ended</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token punctuation">}</span>

size_t ByteStream<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">bytes_written</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token keyword">const</span> <span class="token punctuation">{</span> <span class="token keyword">return</span> _write_count<span class="token punctuation">;</span> <span class="token punctuation">}</span>

size_t ByteStream<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">bytes_read</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token keyword">const</span> <span class="token punctuation">{</span> <span class="token keyword">return</span> _read_count<span class="token punctuation">;</span> <span class="token punctuation">}</span>

size_t ByteStream<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">remaining_capacity</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token keyword">const</span> <span class="token punctuation">{</span> <span class="token keyword">return</span> _capacity <span class="token operator">-</span> _buffer<span class="token punctuation">.</span><span class="token function">size</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token punctuation">}</span>
</code></pre>
<p>extra:<br>
因特网本身并不提供对传输“可靠性”的保证，而是由客户端和服务器上的操作系统来完成这个任务。我们现在要实现的是位于TCP连接两端的套接字中的字节流数据结构。TCPSocket中有两个ByteStream，inbound用于接收数据，outbound用于发送数据。 大体上，ByteStream具有一定的容量，最大允许存储该容量大小的数据；在读取端读出一部分数据后，它会释放掉已经被读出的内容，以腾出空间继续让写端写入数据。<br>
字节流以类ByteStream来实现，byte_stream.hh中声明了这个类，以及它的内部变量和成员函数；各个成员函数在 byte_stream.cc中予以实现。<br>
Someone:(来自知乎)<br>
我最开始使用的方法是创建一个固定长度的string，然后通过索引值进行循环读写（下标索引本质还是指针，所以也不符合现代C++的要求），通过了lab0的测试；但是在lab1中发现一直因为超时而无法通过fsm_stream_reassembler_many这个测试用例。在花了大把时间后发现原因在于此处通过索引值对传入的字节流进行拷贝的方式效率太低！后来自己写了个小程序测试，果然如此。在C++中，虽然string的各种方法一般被认为会比普通数组的操作要慢，但字符串的拼接是一个例外。经实验和上网搜索验证，string的+拼接的确要比按索引值逐个复制要快。我之前一直想当然地以为string的拼接是基于按索引复制，虽然现在还是不清楚内部究竟如何实现，但先记住这个事实。</p>
<h1 id="reference">reference</h1>
<ol>
<li><a href="https://hexyoungs.club/blog/cs144-lab0/">CS144 Lab0 学习笔记</a></li>
<li><a>CS144计算机网络lab0：networking热身</a></li>
<li><a href="https://www.cnblogs.com/kangyupl/p/stanford_cs144_labs.html">【计算机网络】Stanford CS144 Lab Assignments 学习笔记</a></li>
<li><a href="https://cs144.github.io/doc/lab0/class_t_c_p_socket.html">TCPSocket的示例代码</a></li>
<li>看到下划线延伸出的<a href="https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/naming/">变量命名</a></li>
<li><a href="https://www.jianshu.com/p/3aa3e592fb35">git使用删除remote</a></li>
<li><a href="https://blog.csdn.net/Lucky_LXG/article/details/77849212">git push</a></li>
</ol>

