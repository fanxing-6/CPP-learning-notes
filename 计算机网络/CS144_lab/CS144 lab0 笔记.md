# CS144 lab0 笔记

下学期就学习计算机网络了,假期正好找个`lab`预习一下

## 配置

直接用 WSL2 + Clion(安装在WSL2上) 做的实验,还是比用vscode方便一些的

1. 直接fork仓库
2. git clone ...
3. 用clion打开,他都会自动生成
4. 开始写代码

![image-20220122221304319](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201222213470.png)

## 实验 writing webget(入门)

这个实验就是让你熟悉一下网络编程,做之前最好读一下官方推荐的文档

```cpp
void get_URL(const string &host, const string &path) {

    TCPSocket sock1;
    sock1.connect(Address(host, "http"));
    sock1.write("GET " + path + " HTTP/1.1\r\n" + "Host: " + host + "\r\n" + "Connection: close\r\n\r\n");
    sock1.shutdown(SHUT_WR); //关闭写,告诉服务器不用等着了
    while (!sock1.eof()) {
        cout << sock1.read();
    }
    sock1.close();// 别忘了关掉

}
```

## 实现一个可靠的比特流(简单)

实现一个`buffer`可以一边读一边写

- `buffer`的大小不变

- 满了就不能再写了
- 读到`eof`就不能读了
- 不需要考虑多线程



![image-20220122223233604](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201222232719.png)

本来是采用`std::deque`实现的,后来发现链表方式效率更高一些

*byte_stream.hh*

```cpp
class ByteStream {
  private:
    bool _error = false;  //!< Flag indicating that the stream suffered an error.
    bool _input_ended = false;
    size_t _capacity;
    size_t _buffer_size = 0;    // capacity中已经有的字符大小
    size_t _bytes_written = 0;  // 计数器,记录一共写入多少
    size_t _bytes_read = 0;     // 计数器,记录一共读取多少
    std::list<char> _stream{};  // stream
  public:
```

*byte_stream.cc*

```cpp
#include "byte_stream.hh"

// Dummy implementation of a flow-controlled in-memory byte stream.

// For Lab 0, please replace with a real implementation that passes the
// automated checks run by `make check_lab0`.

// You will need to add private members to the class declaration in `byte_stream.hh`

template <typename... Targs>
void DUMMY_CODE(Targs &&... /* unused */) {}

using namespace std;
ByteStream::ByteStream(const size_t capacity) : _capacity(capacity) {}

size_t ByteStream::write(const string &data) {
    size_t write_count = 0;
    for (const char c : data) {
        // not very efficient to do conditional in loop
        if (_capacity - _buffer_size <= 0)
            break;
        else {
            _stream.push_back(c);
            ++_buffer_size;
            ++_bytes_written;
            ++write_count;
        }
    }

    return write_count;
}

//! \param[in] len bytes will be copied from the output side of the buffer
string ByteStream::peek_output(const size_t len) const {
    const size_t peek_length = len > _buffer_size ? _buffer_size : len;
    auto it = _stream.begin();
    advance(it, peek_length);
    return string(_stream.begin(), it);
}

//! \param[in] len bytes will be removed from the output side of the buffer
void ByteStream::pop_output(const size_t len) {
    size_t pop_length = len > _buffer_size ? _buffer_size : len;
    _bytes_read += pop_length;
    _buffer_size -= pop_length;
    while (pop_length--)
        _stream.pop_front();
}

//! Read (i.e., copy and then pop) the next "len" bytes of the stream
//! \param[in] len bytes will be popped and returned
//! \returns a string
std::string ByteStream::read(const size_t len) {
    const string result = peek_output(len);
    pop_output(len);
    return result;
}

void ByteStream::end_input() { _input_ended = true; }

bool ByteStream::input_ended() const { return _input_ended; }

size_t ByteStream::buffer_size() const { return _buffer_size; }

bool ByteStream::buffer_empty() const { return _stream.size() == 0; }

bool ByteStream::eof() const { return _input_ended && buffer_empty(); }

size_t ByteStream::bytes_written() const { return _bytes_written; }

size_t ByteStream::bytes_read() const { return _bytes_read; }

size_t ByteStream::remaining_capacity() const { return _capacity - _buffer_size; }
```

![image-20220122223716975](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201222237016.png)

