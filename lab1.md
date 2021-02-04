
> https://kangyupl.gitee.io/cs144.github.io/assignments/lab1.pdf
# Lab1


# 1 overview
This TCP implementation managed to produce a pair of reliable in-order byte streams (one from you to the server, and one in the opposite direction), even though the underlying network only delivers “best-effort” datagrams. 

By this we mean: short packets of data that can be lost, reordered, altered, or duplicated. You also implemented the byte-stream abstraction yourself, in memory within one computer. 
（对上个lab的回顾）

# 2 get started


# 3 put substrings in sequence
> In this and the next lab, you will implement a **TCP receiver**: the module that receives datagrams and turns them into a reliable byte stream to be read by the user (just as your webget program read the byte stream from the webserver in Lab 0).

 > In this lab you’ll write the **data structure that will be responsible for this reassembly**: a StreamReassembler. It will receive substrings (consisting of a string of bytes, and the index of the first byte of the string within the larger stream) and supply a ByteStream with all of the data correctly ordered.
 
要求实现一个流重组器（stream reassembler），可以将带索引的字节流碎片重组成有序的字节流。
每个字节流碎片都通过**索引、长度、内容**三要素进行描述。重组完的字节流应当被送入指定的字节流（byte stream）对象`_output`中。
>注意事项：[LAB1#](https://www.cnblogs.com/kangyupl/p/stanford_cs144_labs.html#1029644895)
>思路参考： [CS144计算机网络lab1：组装子串](https://zhuanlan.zhihu.com/p/262274265)

> 分析：使用set STL 自带红黑树性质
C++中set这个数据结构封装了红黑树，后者在查找、删除和插入上的效率比较高；set还适用于此的地方在于它在插入新元素时会按“大小”关系对元素进行排序，我们可以通过迭代器的增减按顺序访问元素，这在判断新子串是否可以和索引位置上在附近的子串进行合并时非常有用。若set中的元素不是基本数据类型，我们需要对元素的大小关系进行定义，在这里我们通过重载关于结构体的“<”操作符来实现。结构体中存储的则是各个子串的数据和首字节索引。

![enter image description here](https://pic2.zhimg.com/80/v2-54e61b6e7639e7270e5a2e5fb39d35d9_720w.jpg)
分析：
- 蓝色 已读取
绿色 在队列中排好序 未读取
红色 剩余bytestream容量

- 若新接收到的字符串全部位于蓝色区域或者first unacceptable之后的区域，则会直接被StreamReassembler丢弃。
- push_substring 接受数据流，（主要工作只有这一个）大致思路：
  * 如果接收到的新子串不符合要求则直接丢弃，否则若可以写入则直接写，若不能则存放在一边的缓冲区；
  * 存放在缓冲区中的子串要按它们在大字符串中的位置进行排序，存放前中要考察新串能否与已有的各个子串进行合并，因为会有交叉的情况出现；
  * 如果发现新串在原有的子串中已经出现或者被它们包含，则亦丢弃；若新串直接写入，还要考察缓冲区中的子串是否可以继续写入。
-  merge_block 合并并排序数据 ( 辅助函数)

这次参考的代码主要是知乎上的[CS144计算机网络lab1：组装子串](https://zhuanlan.zhihu.com/p/262274265)

ps：有一个逻辑注意，组装好的都是连续且排好序的，缓冲区是分散在等待组装的数据流

stream_reassembler.hh
```cpp
// struct node 定义 **索引、长度、内容**
private: 
	struct Node{
        std::string data;
        size_t index;
        Node(std::string s, size_t x) : data(s), index(x) {}
        // 这里语法
        bool operator<(const struct Node b) const {
            return this->index < b.index;
        } 
    };
	ByteStream _output;  //!< The reassembled in-order byte stream
    size_t _capacity;    //!< The maximum number of bytes
    size_t _bytes_unassembled;
    std::set<struct Node> _substr_waiting; //set.insert 用法
    bool _flag_eof;
    size_t _pos_eof;
```

两份代码区别，[直接初始化 默认初始化](https://www.cnblogs.com/lustar/p/7450097.html)
[Relational operators](https://en.cppreference.com/w/cpp/language/operators)
[C++的重载操作符（operator）介绍](https://blog.csdn.net/liitdar/article/details/80654324)
[c++ set使用](http://www.cplusplus.com/reference/set/set/insert/)可直接插入struct node,  插入后自动排序
[substr_lower_bound()](https://www.geeksforgeeks.org/set-lower_bound-function-in-c-stl/)

```ruby
横着看
stream_start		(0)first_unread	first_unsembled	   unaccept
v                   v				v				   v
+------------------------------------------------------+--------
|                	|				|		   		   |  out	
+------------------------------------------------------+--------
已经读取				排列好未读取			缓冲区未整理		无法接收
					_substr_waiting	  _bytes_unassembled
```


stream_reassembler.cc
```c++
// Construct a `StreamReassembler` that will store up to `capacity` bytes.
// 传进capacity
// 根据capacity 修改StreamReassembler，储存数据容器的大小
StreamReassembler(const size_t capacity);

//辅助函数：向缓冲区写入(整理)
void StreamReassembler::insert_substr_waiting(const struct Node &node)
如果等待区是空
	插入数据
	记录未排序数据的大小

记录node temp = node （const）
若node的左边有节点，考察是否能与左边的节点合并（用了迭代器）

// Receive a substring and write any newly contiguous bytes into the stream. 
// 传进一个 带着index标记的data
void push_substring(const string &data, const uint64_t index, const bool eof);
接收传进的数据
 ·超出不作处理
 · 留下能处理的部分
 ·看新子串能否直接写入
  · 新子串写完后能否写入缓冲区的
 ·若新子串不能写入，则存放在缓冲区


// `data`: the segment 
// `index` indicates the index (place in sequence) of the first byte in `data` 
// `eof`: the last byte of this segment is the last byte in the entire stream


// Access the reassembled byte stream
ByteStream &stream_out();


// The number of bytes in the substrings stored but not yet reassembled
size_t unassembled_bytes() const;

// Is the internal state empty (other than the output stream)?
bool empty() const;
```

ps： 由于新课程代码不同，小心替换cmakelist文件和代码文件
> 可恶的小插曲浪费了两个小时，不过也明白了cmake的作用。cap 文件缺失， 添加后要在cmake 文件中增加一行代码
# reference
1. [LAB1#](https://www.cnblogs.com/kangyupl/p/stanford_cs144_labs.html#1029644895)
2.  [CS144计算机网络lab1：组装子串](https://zhuanlan.zhihu.com/p/262274265)

