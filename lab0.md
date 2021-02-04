>  https://kangyupl.gitee.io/cs144.github.io/assignments/lab0.pdf
>  一些过程略

## 2.1 Fetch a Web page
**这里是http request 的步骤**
1. `telnet cs144.keithw.org http `. This tells the telnet program to open a reliable byte stream between your computer and another computer (named cs144.keithw.org), and with a particular service running on that computer: the “http” service, for the Hyper-Text Transfer Protocol, used by the World Wide Web.
2.  `GET /hello HTTP/1.1` the path part
3.  `Host: cs144.keithw.org` the host part
4.  Hit the Enter key one more time: . This tells the server that you are done with your HTTP request.

## 2.2 Send yourself an email
jump

## 2.3 Listening and connecting
1. In one terminal window, run  `netcat -v -l -p 9090 ` on your VM. 
2. Leave netcat running. In another terminal window, run `telnet localhost 9090` (also on your VM).



# 3 Writing a network program using an OS stream socket
> https://zhuanlan.zhihu.com/p/37633767

write a short program that fetches a Web page over the Internet: create a reliable bidirectional in-order byte stream between two programs, one running on your computer, and the other on a different computer across the Internet.
This feature is known as a stream socket.

cmake的意义：



## 3.4 Writing webget
>  [CS144计算机网络lab0：networking热身](https://zhuanlan.zhihu.com/p/257283830)
>  [【计算机网络】Stanford CS144 Lab Assignments 学习笔记](https://www.cnblogs.com/kangyupl/p/stanford_cs144_labs.html)
>   [TCPSocket的示例代码](https://cs144.github.io/doc/lab0/class_t_c_p_socket.html)

>分析：让我自己写是肯定写不出来的，先分析一遍答案，再写
>要是说的话 就是和2.1 是一样的

```c++
void get_URL(const string &host, const string &path) {

    TCPSocket sock{};
    sock.connect(Address(host,"http"));  //host
    sock.write("GET "+path+" HTTP/1.1\r\nHost: "+host+"\r\n\r\n"); // the path+ hit one more time
    sock.shutdown(SHUT_WR);
    while(!sock.eof()){
        cout<<sock.read();
    }
    sock.close();
    return;
}
```
attention:
注意在客户端socket发送完请求报文之后，要使用shutdown方法破坏客户端和服务器套接字之间的连接；如果仅用close方法，则只是关闭socket连接的某一个标识，连接本身还是开着的（我不太确定Connection: close\r\n\r\n和后面的socket.shutdown作用是否重复，不深究）。
套接字本身相当于一个文件描述符，程序通过它对某个“文件”（字节流抽象）进行写入和读出的操作。我们通过TCPSocket类中的eof方法判断套接字中写入的数据是否已经到达了EOF，只要还没有，就仍要使用read方法进行读取（在下一个小实验中，我们会更加清楚为什么要这么做）。


# 4 An in-memory reliable byte stream
> [CS144 Lab0 学习笔记](https://hexyoungs.club/blog/cs144-lab0/)
> [CS144计算机网络lab0：networking热身](https://zhuanlan.zhihu.com/p/257283830)
> [C++中queue和deque的区别](https://zhuanlan.zhihu.com/p/77981148)
>  [size_t](https://en.cppreference.com/w/cpp/types/size_t)
>  [为什么size_t重要？（Why size_t matters）](https://link.zhihu.com/?target=https%3A//jeremybai.github.io/blog/2014/09/10/size-t)
>  [string 语法](http://www.cplusplus.com/reference/string/string/assign/)
>  [deque语法](http://www.cplusplus.com/reference/deque/deque/)


第二个任务要求实现一个有序字节流类（in-order byte stream），使之支持读写、容量控制。由于是单线程，并且不考虑并发读写，我们只需要利用 cpp 标准库里面的容器(这里用了 dequeue )保存数据，然后适配所有的接口并且通过测试用例即可。
这个字节流类似于一个带容量的队列，从一头读，从另一头写。当流中的数据达到容量上限时，便无法再写入新的数据。特别的，写操作被分为了peek和pop两步。
peek为从头部开始读取指定数量的字节，pop为弹出指定数量的字节。

（看着这块自己写代码，不要看答案）逻辑：
1. 写操作，如果数据大于剩余空闲空间大小，则只等于最大剩余容量
2. 记录数量
3. 向 buffer  里写入数据
```ruby
+-------------------+  <- capacity
|                   |
|                   |
|                   |
+-------------------+  <- buffer.size()
|					|
+-------------------+  <- 0
```
```c++
// .hh 命名参数 
class ByteStream {
  private:
    // Your code here -- add private members as necessary.
    std::deque<char> _buffer = {};
    size_t _capacity = 0;
    size_t _read_count = 0;
    size_t _write_count = 0;
    bool _input_ended_flag = false;
    bool _error = false;  //!< Flag indicating that the stream suffered an error.
    //......

// .cc 文件 程序
ByteStream::ByteStream(const size_t capacity) : _capacity(capacity) {} // 这里为什么？

size_t ByteStream::write(const string &data) {
    size_t len = data.length(); // 命名是加上变量类型
    // 这块记不住加不加括号，现默认都加上
    if (len > _capacity - _buffer.size()) {
        len = _capacity - _buffer.size();
    }
    _write_count += len;
    for (size_t i = 0; i < len; i++) {
        _buffer.push_back(data[i]);
    }
    return len;
}

//! \param[in] len bytes will be copied from the output side of the buffer
string ByteStream::peek_output(const size_t len) const {
    size_t length = len; // 他这里用了变量获取常量值，方便下面操作
    if (length > _buffer.size()) {
        length = _buffer.size();
    }
    return string().assign(_buffer.begin(), _buffer.begin() + length);
}

//! \param[in] len bytes will be removed from the output side of the buffer
void ByteStream::pop_output(const size_t len) {
    size_t length = len;
    if (length > _buffer.size()) {
        length = _buffer.size();
    }
    _read_count += length;  
    // 这里涉及到了 _read_count,读取数量相当于出战的数量
    while (length--) {
        _buffer.pop_front();
    }
    return;
}

void ByteStream::end_input() { _input_ended_flag = true; }

bool ByteStream::input_ended() const { return _input_ended_flag; }

size_t ByteStream::buffer_size() const { return _buffer.size(); }

bool ByteStream::buffer_empty() const { return _buffer.size() == 0; }

bool ByteStream::eof() const { return buffer_empty() && input_ended(); }

size_t ByteStream::bytes_written() const { return _write_count; }

size_t ByteStream::bytes_read() const { return _read_count; }

size_t ByteStream::remaining_capacity() const { return _capacity - _buffer.size(); }
```


extra:
因特网本身并不提供对传输“可靠性”的保证，而是由客户端和服务器上的操作系统来完成这个任务。我们现在要实现的是位于TCP连接两端的套接字中的字节流数据结构。TCPSocket中有两个ByteStream，inbound用于接收数据，outbound用于发送数据。 大体上，ByteStream具有一定的容量，最大允许存储该容量大小的数据；在读取端读出一部分数据后，它会释放掉已经被读出的内容，以腾出空间继续让写端写入数据。
字节流以类ByteStream来实现，byte_stream.hh中声明了这个类，以及它的内部变量和成员函数；各个成员函数在 byte_stream.cc中予以实现。
Someone:(来自知乎)
我最开始使用的方法是创建一个固定长度的string，然后通过索引值进行循环读写（下标索引本质还是指针，所以也不符合现代C++的要求），通过了lab0的测试；但是在lab1中发现一直因为超时而无法通过fsm_stream_reassembler_many这个测试用例。在花了大把时间后发现原因在于此处通过索引值对传入的字节流进行拷贝的方式效率太低！后来自己写了个小程序测试，果然如此。在C++中，虽然string的各种方法一般被认为会比普通数组的操作要慢，但字符串的拼接是一个例外。经实验和上网搜索验证，string的+拼接的确要比按索引值逐个复制要快。我之前一直想当然地以为string的拼接是基于按索引复制，虽然现在还是不清楚内部究竟如何实现，但先记住这个事实。

# reference
1. [CS144 Lab0 学习笔记](https://hexyoungs.club/blog/cs144-lab0/)
2.  [CS144计算机网络lab0：networking热身](htps://zhuanlan.zhihu.com/p/257283830)
3.  [【计算机网络】Stanford CS144 Lab Assignments 学习笔记](https://www.cnblogs.com/kangyupl/p/stanford_cs144_labs.html)
4. [TCPSocket的示例代码](https://cs144.github.io/doc/lab0/class_t_c_p_socket.html)
5. 看到下划线延伸出的[变量命名](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/naming/)
6. [git使用删除remote](https://www.jianshu.com/p/3aa3e592fb35)
7. [git push](https://blog.csdn.net/Lucky_LXG/article/details/77849212)
