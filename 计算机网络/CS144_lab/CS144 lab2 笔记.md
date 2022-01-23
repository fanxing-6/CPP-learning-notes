# CS144 lab2 笔记

## 介绍

在`lab0`中，我们实现了一个`ByteStream`。

在`lab1`中，实现了一个重组字符片段的`StreamReassembler`，重组收到的字符片段，并且将排序好的字符串退送到`ByteStream`

在`lab2`中，j将实现一个` TCPReceiver`,它将在`TCP segments`和`byte stream`之间进行转换

![image-20220123222702799](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201232227902.png)

通过这个图片

- `ackno`就是第一个未排序片段的索引,是期望下一个收到的片段索引

- 第一个未排序片段与流末端索引之间的距离就是`window size`(TCP窗口)

  *the distance between the “first unassembled” index and the “first unacceptable” index.
  This is called the “window size”*

## Translating between 64-bit indexes and 32-bit seqnos

![img](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201232234807.png)

1. 通过`TCP`报头我们可以知道,传输过程中的`seq`是32位的,但我们本地的`seq`是64位系统下的,所以我们需要将`seq(64bit)--->seq(32bit)`,32位最大值为 2^32^-1,超过这个数字就从0开始

2. **TCP seq 以一个32为随机值初始化**:这个目的是为了防止被猜到,以及网络中较早的数据报造成干扰,一端连接中第一个`seq`就以一个32位的数字初始化,叫做`Initial Sequence Number(ISN)`,之后每个seq/mod 2^32^
3. **连接开始和结束每个占用一个序列号**：除了确保收到所有字节的数据，TCP确保流的开始和结束同样是是可靠的。因此，在TCP中，SYN（流开始）和FIN（流终端）控制标志被分配序列号。都占据一个序列号。 （SYN标志占用的序列号就是ISN。）流中的每个数据字节还占用一个序列号。请记住，SYN和FIN不是流本身的一部分，而不是“字节” ---它们代表字节流本身的开始和结束。

这些seq在每个TCP段的头中发送。**绝对序列号:**始终以零开始并且不包装(就是64位)，**流索引:**`StreamReassEmbler`流中的每个字节的索引，从零开始,64bit,具体见下图:

![image-20220123224747548](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201232247594.png)

### 思路

- 很显然这种转换不是唯一的——seqno每次增加$2^{32}$值都不变，但是absolute seqno变化。为了确定唯一的结果，我们需要`checkpoint`，即将可能的结果中距离`checkpoint`最近的作为最终结果。

- `checkpoint`表示最近一次转换求得的`absolute seqno`，而本次转换出的`absolute seqno`应该选择与上次值最为接近的那一个。原理是虽然segment不一定按序到达，但几乎不可能出现相邻到达的两个segment序号差值超过`INT32_MAX`的情况
- 如果想不出来,就在纸上画一画就能想出来了

### 实现

```C++
//! Transform an "absolute" 64-bit sequence number (zero-indexed) into a WrappingInt32
//! \param n The input absolute 64-bit sequence number
//! \param isn The initial sequence number
WrappingInt32 wrap(uint64_t n, WrappingInt32 isn) { return WrappingInt32(static_cast<uint32_t>(n) + isn.raw_value()); }

//! Transform a WrappingInt32 into an "absolute" 64-bit sequence number (zero-indexed)
//! \param n The relative sequence number
//! \param isn The initial sequence number
//! \param checkpoint A recent absolute 64-bit sequence number
//! \returns the 64-bit sequence number that wraps to `n` and is closest to `checkpoint`
//!
//! \note Each of the two streams of the TCP connection has its own ISN. One stream
//! runs from the local TCPSender to the remote TCPReceiver and has one ISN,
//! and the other stream runs from the remote TCPSender to the local TCPReceiver and
//! has a different ISN.
uint64_t unwrap(WrappingInt32 n, WrappingInt32 isn, uint64_t checkpoint) {
    int32_t interval = n - wrap(checkpoint, isn);
    int64_t result = checkpoint + interval;
    if (result >= 0)
        return result;
    else
        return result + (1ul << 32);
}

```



## Implementing the TCP receiver

对于接收端,在这个实验中只需要处理示意图中彩色部分:

![image-20220123231716604](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201232317654.png)

在做实验之前最好看一下`TCPSegment`的相关实现

### 思路

主要就是参考下图中TCP的过程,将其分为图中三种情况,一一实现即可,错误不需要考虑,将在之后的实验进行处理

![image-20220123232228395](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201232322444.png)

#### segment received()

- 请设置初始序列号。设置ISN,并且不要忘记使用 。请注意，SYN标志只是标题中的一个标志。相同的段也可以携带数据，甚至可以设置FIN标志。所以，即使受到syn也不能抛弃他的片段以及fin
- 将任何数据或流终端标记推向StreamReasseMbler。如果FIN标志设置在TCPSegment的标题中，这意味着有效载荷的最后一个字节是整个流的最后一个字节。

#### ackno()

- 返回包含接收器尚未知道的第一个字节的序列号的`wraxingInt32`。这是Window的左边缘。如果尚未设置ISN，则返回一个空可选

#### window size()

- 见代码

### 实现

*tcp_receiver.hh*

```c++
class TCPReceiver {
    //! Our data structure for re-assembling bytes.
    StreamReassembler _reassembler;

    //! The maximum number of bytes we'll store.
    size_t _capacity;
    bool _syn {false};
    WrappingInt32 _isn{0};
```

*tcp_receiver.cc*

```c++
void TCPReceiver::segment_received(const TCPSegment &seg) {
    const TCPHeader &_tcp_header = seg.header();
    /*
     * 不需要考虑很多,直接按照tcp接收端状态转换图写就行,其他情况都可以交给reassembler处理
     */
    if (!_syn)  // 如果未收到过syn
    {
        if (!_tcp_header.syn)// 如果包中含有syn就继续,没有就直接返回,抛弃包
            return;
        _syn = true;
        _isn = _tcp_header.seqno;
    }
    // ack 期望下一个收到的片段索引
    uint64_t _ackno = _reassembler.stream_out().bytes_written() + 1;
    // 计算 seq
    uint64_t _seqno = unwrap(_tcp_header.seqno, _isn, _ackno);
    // 注意syn也占用seqno,所以别忘了
    uint64_t _index = _seqno - 1 + static_cast<uint64_t>(_tcp_header.syn);
    _reassembler.push_substring(seg.payload().copy(), _index, _tcp_header.fin);
}

optional<WrappingInt32> TCPReceiver::ackno() const {
    if (!_syn) // 如果未建立连接,直接返回空
        return nullopt;
    uint64_t _ackno = _reassembler.stream_out().bytes_written() + 1;
    if (_reassembler.stream_out().input_ended())// 如果结束连接,返回的ack要算上sender发来的fin
        _ackno++;
    return WrappingInt32(_isn) + _ackno;
}
// 如下
size_t TCPReceiver::window_size() const { return _capacity - _reassembler.stream_out().buffer_size(); }

```





