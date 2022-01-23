# CS144 lab1 笔记

![](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201222338798.png)



<img src="https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201222256243.png" alt="image-20220122225625163" style="zoom:50%;" />

上图是TCP实现中模块和数据流的安排,我们要实现的就是`StreamReassembler`

一个字符重组器,将乱序的字符串,按照索引排序,使其成为连续字符,供`TCPSender`和`TCPReceiver`使用

- 有容量限制,超出的字符直接丢掉(不是整个片段)

- TCP接收到的片段从零开始,不会溢出
- 任何报文段,只要排好序(重组,去除重叠),就必须放入`ByteStream`
- 收到`eof`表示这个字符流结束了,并且这个字符串为最后一个,但不是现在停止接收,必须等到`eof`及之前的字符片段全部排序完成并且实现之后才能终止字符流的写入(因为乱序写入)
- `_output`对应的是`width`区域,已读范围不属于`_output`,这里的变化已经在上一个实验处理好

<img src="https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201222357707.png" alt="image-20220122235717632" style="zoom:50%;" />

<img src="https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201222358832.png" alt="image-20220122235804651" style="zoom:50%;" />

## 思路:

- 用 set 来暂存报文段，按照报文段的 index 大小对比重载 < 运算符。
- 收到新片段时根据`Index`和`length`并且根据下图重叠情况进行合并,切除

![image-20220123001615050](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201230016132.png)

- 合并后,遍历片段,如果有能与个前面已排序片段合并的就合并,经合并的片段放入`ByteStream`中
- 如果`eof`为真,并且全部排序时,停止`ByteStream`输入
- 重复上述过程

## 代码

*stream_reassembler.hh*

```C++
class StreamReassembler {
  private:
    // Your code here -- add private members as necessary.

    ByteStream _output;  //!< The reassembled in-order byte stream
    size_t _capacity;    //!< The maximum number of bytes
    size_t _first_unread = 0;
    size_t _first_unassembled = 0;
    size_t _first_unacceptable;
    bool _eof = false;
    struct seg {
        size_t index;
        size_t length;
        std::string data;
        bool operator<(const seg& t) const { return index < t.index; }
    };
    std::set<seg> _stored_segs = {};

    void _add_new_seg(seg &new_seg, bool eof);
    void _handle_overlap(seg &new_seg);
    void _stitch_output();
    void _stitch_one_seg(const seg &new_seg);
    void _merge_seg(seg &new_seg, const seg &other);

  public:
```



**stream_reassembler.cc*

```C++
#include "stream_reassembler.hh"

// Dummy implementation of a stream reassembler.

// For Lab 1, please replace with a real implementation that passes the
// automated checks run by `make check_lab1`.

// You will need to add private members to the class declaration in `stream_reassembler.hh`

template <typename... Targs>
void DUMMY_CODE(Targs &&... /* unused */) {}

using namespace std;
StreamReassembler::StreamReassembler(const size_t capacity)
    : _output(capacity), _capacity(capacity), _first_unacceptable(capacity) {}

//! \details This function accepts a substring (aka a segment) of bytes,
//! possibly out-of-order, from the logical stream, and assembles any newly
//! contiguous substrings and writes them into the output stream in order.
void StreamReassembler::push_substring(const string &data, const size_t index, const bool eof) {
    _first_unread = _output.bytes_read();
    _first_unacceptable = _first_unread + _capacity;
    seg new_seg = {index, data.length(), data};
    _add_new_seg(new_seg, eof);
    _stitch_output();
    if (empty() && _eof)
        _output.end_input();
}

void StreamReassembler::_add_new_seg(seg &new_seg, const bool eof) {
    // check capacity limit, if unmeet limit, return
    // cut the bytes in NEW_SEG that will overflow the _CAPACITY
    // note that the EOF should also be cut
    // cut the bytes in NEW_SEG that are already in _OUTPUT
    // _HANDLE_OVERLAP()
    // update _EOF
    if (new_seg.index >= _first_unacceptable)
        return;
    bool eof_of_this_seg = eof;
    if (int overflow_bytes = new_seg.index + new_seg.length - _first_unacceptable; overflow_bytes > 0) {
        int new_length = new_seg.length - overflow_bytes;
        if (new_length <= 0)
            return;
        eof_of_this_seg = false;
        new_seg.length = new_length;
        new_seg.data = new_seg.data.substr(0, new_seg.length);
    }
    if (new_seg.index < _first_unassembled) {
        int new_length = new_seg.length - (_first_unassembled - new_seg.index);
        if (new_length <= 0)
            return;
        new_seg.length = new_length;
        new_seg.data = new_seg.data.substr(_first_unassembled - new_seg.index, new_seg.length);
        new_seg.index = _first_unassembled;
    }
    _handle_overlap(new_seg);
    // if EOF was received before, it should remain valid
    _eof = _eof || eof_of_this_seg;
}

void StreamReassembler::_handle_overlap(seg &new_seg) {
    for (auto it = _stored_segs.begin(); it != _stored_segs.end();) {
        auto next_it = ++it;
        --it;
        if ((new_seg.index >= it->index && new_seg.index < it->index + it->length) ||
            (it->index >= new_seg.index && it->index < new_seg.index + new_seg.length)) {
            _merge_seg(new_seg, *it);
            _stored_segs.erase(it);
        }
        it = next_it;
    }
    _stored_segs.insert(new_seg);
}

void StreamReassembler::_stitch_output() {
    // _FIRST_UNASSEMBLED is the expected next index_FIRST_UNASSEMBLED
    // compare _STORED_SEGS.begin()->index with
    // if equals, then _STITCH_ONE_SEG() and erase this seg from set
    // continue compare until not equal or empty
    while (!_stored_segs.empty() && _stored_segs.begin()->index == _first_unassembled) {
        _stitch_one_seg(*_stored_segs.begin());
        _stored_segs.erase(_stored_segs.begin());
    }
}

void StreamReassembler::_stitch_one_seg(const seg &new_seg) {
    // write string of NEW_SEG into _OUTPUT
    // update _FIRST_UNASSEMBLED
    _output.write(new_seg.data);
    _first_unassembled += new_seg.length;
    // both way of updating _FIRST_UNASSEMBLED is ok
    // _first_unassembled = _output.bytes_written();
}

void StreamReassembler::_merge_seg(seg &new_seg, const seg &other) {
    size_t n_index = new_seg.index;
    size_t n_end = new_seg.index + new_seg.length;
    size_t o_index = other.index;
    size_t o_end = other.index + other.length;
    string new_data;
    if (n_index <= o_index && n_end <= o_end) {
        new_data = new_seg.data + other.data.substr(n_end - o_index, n_end - o_end);
    } else if (n_index <= o_index && n_end >= o_end) {
        new_data = new_seg.data;
    } else if (n_index >= o_index && n_end <= o_end) {
        new_data =
            other.data.substr(0, n_index - o_index) + new_seg.data + other.data.substr(n_end - o_index, n_end - o_end);
    } else{
        new_data = other.data.substr(0, n_index - o_index) + new_seg.data;
    }
    new_seg.index = n_index < o_index ? n_index : o_index;
    new_seg.length = (n_end > o_end ? n_end : o_end) - new_seg.index;
    new_seg.data = new_data;
}

size_t StreamReassembler::unassembled_bytes() const {
    size_t unassembled_bytes = 0;
    for (auto it = _stored_segs.begin(); it != _stored_segs.end(); ++it)
        unassembled_bytes += it->length;
    return unassembled_bytes;
}

bool StreamReassembler::empty() const { return unassembled_bytes() == 0; }

```

![image-20220123004414481](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201230044517.png)

