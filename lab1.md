---


---

<blockquote>
<p><a href="https://kangyupl.gitee.io/cs144.github.io/assignments/lab1.pdf">https://kangyupl.gitee.io/cs144.github.io/assignments/lab1.pdf</a></p>
</blockquote>
<h1 id="lab1">Lab1</h1>
<h1 id="overview">1 overview</h1>
<p>This TCP implementation managed to produce a pair of reliable in-order byte streams (one from you to the server, and one in the opposite direction), even though the underlying network only delivers “best-effort” datagrams.</p>
<p>By this we mean: short packets of data that can be lost, reordered, altered, or duplicated. You also implemented the byte-stream abstraction yourself, in memory within one computer.<br>
（对上个lab的回顾）</p>
<h1 id="get-started">2 get started</h1>
<h1 id="put-substrings-in-sequence">3 put substrings in sequence</h1>
<blockquote>
<p>In this and the next lab, you will implement a <strong>TCP receiver</strong>: the module that receives datagrams and turns them into a reliable byte stream to be read by the user (just as your webget program read the byte stream from the webserver in Lab 0).</p>
</blockquote>
<blockquote>
<p>In this lab you’ll write the <strong>data structure that will be responsible for this reassembly</strong>: a StreamReassembler. It will receive substrings (consisting of a string of bytes, and the index of the first byte of the string within the larger stream) and supply a ByteStream with all of the data correctly ordered.</p>
</blockquote>
<p>要求实现一个流重组器（stream reassembler），可以将带索引的字节流碎片重组成有序的字节流。<br>
每个字节流碎片都通过<strong>索引、长度、内容</strong>三要素进行描述。重组完的字节流应当被送入指定的字节流（byte stream）对象<code>_output</code>中。</p>
<blockquote>
<p>注意事项：<a href="https://www.cnblogs.com/kangyupl/p/stanford_cs144_labs.html#1029644895">LAB1#</a><br>
思路参考： <a href="https://zhuanlan.zhihu.com/p/262274265">CS144计算机网络lab1：组装子串</a></p>
</blockquote>
<blockquote>
<p>分析：使用set STL 自带红黑树性质<br>
C++中set这个数据结构封装了红黑树，后者在查找、删除和插入上的效率比较高；set还适用于此的地方在于它在插入新元素时会按“大小”关系对元素进行排序，我们可以通过迭代器的增减按顺序访问元素，这在判断新子串是否可以和索引位置上在附近的子串进行合并时非常有用。若set中的元素不是基本数据类型，我们需要对元素的大小关系进行定义，在这里我们通过重载关于结构体的“&lt;”操作符来实现。结构体中存储的则是各个子串的数据和首字节索引。</p>
</blockquote>
<p><img src="https://pic2.zhimg.com/80/v2-54e61b6e7639e7270e5a2e5fb39d35d9_720w.jpg" alt="enter image description here"><br>
分析：</p>
<ul>
<li>
<p>蓝色 已读取<br>
绿色 在队列中排好序 未读取<br>
红色 剩余bytestream容量</p>
</li>
<li>
<p>若新接收到的字符串全部位于蓝色区域或者first unacceptable之后的区域，则会直接被StreamReassembler丢弃。</p>
</li>
<li>
<p>push_substring 接受数据流，（主要工作只有这一个）大致思路：</p>
<ul>
<li>如果接收到的新子串不符合要求则直接丢弃，否则若可以写入则直接写，若不能则存放在一边的缓冲区；</li>
<li>存放在缓冲区中的子串要按它们在大字符串中的位置进行排序，存放前中要考察新串能否与已有的各个子串进行合并，因为会有交叉的情况出现；</li>
<li>如果发现新串在原有的子串中已经出现或者被它们包含，则亦丢弃；若新串直接写入，还要考察缓冲区中的子串是否可以继续写入。</li>
</ul>
</li>
<li>
<p>merge_block 合并并排序数据 ( 辅助函数)</p>
</li>
</ul>
<p>这次参考的代码主要是知乎上的<a href="https://zhuanlan.zhihu.com/p/262274265">CS144计算机网络lab1：组装子串</a></p>
<p>ps：有一个逻辑注意，组装好的都是连续且排好序的，缓冲区是分散在等待组装的数据流</p>
<p>stream_reassembler.hh</p>
<pre class=" language-cpp"><code class="prism  language-cpp"><span class="token comment">// struct node 定义 **索引、长度、内容**</span>
<span class="token keyword">private</span><span class="token operator">:</span> 
	<span class="token keyword">struct</span> Node<span class="token punctuation">{</span>
        std<span class="token operator">::</span>string data<span class="token punctuation">;</span>
        size_t index<span class="token punctuation">;</span>
        <span class="token function">Node</span><span class="token punctuation">(</span>std<span class="token operator">::</span>string s<span class="token punctuation">,</span> size_t x<span class="token punctuation">)</span> <span class="token operator">:</span> <span class="token function">data</span><span class="token punctuation">(</span>s<span class="token punctuation">)</span><span class="token punctuation">,</span> <span class="token function">index</span><span class="token punctuation">(</span>x<span class="token punctuation">)</span> <span class="token punctuation">{</span><span class="token punctuation">}</span>
        <span class="token comment">// 这里语法</span>
        <span class="token keyword">bool</span> <span class="token keyword">operator</span><span class="token operator">&lt;</span><span class="token punctuation">(</span><span class="token keyword">const</span> <span class="token keyword">struct</span> Node b<span class="token punctuation">)</span> <span class="token keyword">const</span> <span class="token punctuation">{</span>
            <span class="token keyword">return</span> <span class="token keyword">this</span><span class="token operator">-</span><span class="token operator">&gt;</span>index <span class="token operator">&lt;</span> b<span class="token punctuation">.</span>index<span class="token punctuation">;</span>
        <span class="token punctuation">}</span> 
    <span class="token punctuation">}</span><span class="token punctuation">;</span>
	ByteStream _output<span class="token punctuation">;</span>  <span class="token comment">//!&lt; The reassembled in-order byte stream</span>
    size_t _capacity<span class="token punctuation">;</span>    <span class="token comment">//!&lt; The maximum number of bytes</span>
    size_t _bytes_unassembled<span class="token punctuation">;</span>
    std<span class="token operator">::</span>set<span class="token operator">&lt;</span><span class="token keyword">struct</span> Node<span class="token operator">&gt;</span> _substr_waiting<span class="token punctuation">;</span> <span class="token comment">//set.insert 用法</span>
    <span class="token keyword">bool</span> _flag_eof<span class="token punctuation">;</span>
    size_t _pos_eof<span class="token punctuation">;</span>
</code></pre>
<p>两份代码区别，<a href="https://www.cnblogs.com/lustar/p/7450097.html">直接初始化 默认初始化</a><br>
<a href="https://en.cppreference.com/w/cpp/language/operators">Relational operators</a><br>
<a href="https://blog.csdn.net/liitdar/article/details/80654324">C++的重载操作符（operator）介绍</a><br>
<a href="http://www.cplusplus.com/reference/set/set/insert/">c++ set使用</a>可直接插入struct node,  插入后自动排序<br>
<a href="https://www.geeksforgeeks.org/set-lower_bound-function-in-c-stl/">substr_lower_bound()</a></p>
<pre class=" language-ruby"><code class="prism  language-ruby">横着看
<span class="token function">stream_start</span>		<span class="token punctuation">(</span><span class="token number">0</span><span class="token punctuation">)</span>first_unread	first_unsembled	   unaccept
v                   v				v				   v
<span class="token operator">+</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">+</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span>
<span class="token operator">|</span>                	<span class="token operator">|</span>				<span class="token operator">|</span>		   		   <span class="token operator">|</span>  out	
<span class="token operator">+</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">+</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span><span class="token operator">--</span>
已经读取				排列好未读取			缓冲区未整理		无法接收
					_substr_waiting	  _bytes_unassembled
</code></pre>
<p>stream_reassembler.cc</p>
<pre class=" language-c"><code class="prism ++ language-c"><span class="token comment">// Construct a `StreamReassembler` that will store up to `capacity` bytes.</span>
<span class="token comment">// 传进capacity</span>
<span class="token comment">// 根据capacity 修改StreamReassembler，储存数据容器的大小</span>
<span class="token function">StreamReassembler</span><span class="token punctuation">(</span><span class="token keyword">const</span> size_t capacity<span class="token punctuation">)</span><span class="token punctuation">;</span>

<span class="token comment">//辅助函数：向缓冲区写入(整理)</span>
<span class="token keyword">void</span> StreamReassembler<span class="token punctuation">:</span><span class="token punctuation">:</span><span class="token function">insert_substr_waiting</span><span class="token punctuation">(</span><span class="token keyword">const</span> <span class="token keyword">struct</span> Node <span class="token operator">&amp;</span>node<span class="token punctuation">)</span>
如果等待区是空
	插入数据
	记录未排序数据的大小

记录node temp <span class="token operator">=</span> node （<span class="token keyword">const</span>）
若node的左边有节点，考察是否能与左边的节点合并（用了迭代器）

<span class="token comment">// Receive a substring and write any newly contiguous bytes into the stream. </span>
<span class="token comment">// 传进一个 带着index标记的data</span>
<span class="token keyword">void</span> <span class="token function">push_substring</span><span class="token punctuation">(</span><span class="token keyword">const</span> string <span class="token operator">&amp;</span>data<span class="token punctuation">,</span> <span class="token keyword">const</span> uint64_t index<span class="token punctuation">,</span> <span class="token keyword">const</span> bool eof<span class="token punctuation">)</span><span class="token punctuation">;</span>
接收传进的数据
 ·超出不作处理
 · 留下能处理的部分
 ·看新子串能否直接写入
  · 新子串写完后能否写入缓冲区的
 ·若新子串不能写入，则存放在缓冲区


<span class="token comment">// `data`: the segment </span>
<span class="token comment">// `index` indicates the index (place in sequence) of the first byte in `data` </span>
<span class="token comment">// `eof`: the last byte of this segment is the last byte in the entire stream</span>


<span class="token comment">// Access the reassembled byte stream</span>
ByteStream <span class="token operator">&amp;</span><span class="token function">stream_out</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>


<span class="token comment">// The number of bytes in the substrings stored but not yet reassembled</span>
size_t <span class="token function">unassembled_bytes</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token keyword">const</span><span class="token punctuation">;</span>

<span class="token comment">// Is the internal state empty (other than the output stream)?</span>
bool <span class="token function">empty</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token keyword">const</span><span class="token punctuation">;</span>
</code></pre>
<p>ps： 由于新课程代码不同，小心替换cmakelist文件和代码文件</p>
<blockquote>
<p>可恶的小插曲浪费了两个小时，不过也明白了cmake的作用。cap 文件缺失， 添加后要在cmake 文件中增加一行代码</p>
</blockquote>
<h1 id="reference">reference</h1>
<ol>
<li><a href="https://www.cnblogs.com/kangyupl/p/stanford_cs144_labs.html#1029644895">LAB1#</a></li>
<li><a href="https://zhuanlan.zhihu.com/p/262274265">CS144计算机网络lab1：组装子串</a></li>
</ol>

