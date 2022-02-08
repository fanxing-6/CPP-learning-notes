# CS144 计算机网络实验 lab3 笔记

## 介绍

本实验中,我们将会在之前实验的基础上,实现一个`TCP sender` ----将字节流转换成数据报并发送.

![image-20220124144804905](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201241448010.png)

TCP协议是一个在不可靠的协议上提供可靠的,流量控制的协议。

我们在本实验中会实现一个TCP发送端，负责将发送端应用层传入的比特流转换成一系列由发出的TCP报文段，在另一端，由TCP接收端将TCP报文段转换成原始比特流（也有可能什么都不做，在从未受到syn，或者多次受到的时候），并将`ack`和**窗口**大小返回给TCP发送端



TCP接收端和发送端都负责处理一部分TCP报文段。TCP发送端写入到报文中的每部分都会被TCP接收端解析，包括：seq，SYN，FIN，内容

但是，TCP发送端只会一部分报文的内容，包括：`ackno`和`SYN`

![image-20220124155145910](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201241551986.png)

TCP发送端有以下任务:

- 跟踪`window size`,也就是处理`ackno`和`window sizes`
- 尽可能的填充 ( 自己的 ) `windows`,直到满了或者`ByteStream`空了为止
- 对已经发送但没有收到`ackno`的报文段 ( 未被确认的报文段) 保持跟踪
- 如果超出规定的时间,还没有收到对应的`ack`,就重新发送所有已经发送,但没有收到`ackno`的报文段



#### TCP发送端是如何知道报文段丢失呢?

TCP发送端会发送一系列的报文段. 每个报文段都是来自`ByteStream`的子串加上位置索引`seq`,如果是第一个报文段,需要加上`syn`,最后一个需要加上`fin`

为了发送这些报文段,TCP发送端会对所有已发送的报文段保持跟踪,直到收到响应报文段的`ack`

具体实现:

1. TCP发送端会定期的调用`TCPSender::tick`函数,来表明时间的流逝. 
2. TCP发送端负责监管已发出报文段,如果最早的已发出但没有收到`ack`的报文段,超出了规定时间,它将会被重新发送

#### 实现记录时间,以及计算时间

1. 每隔几毫秒,TCP发送端的`TCPSender’s tick`方法将被调用，它告自上次调用此方法以来已经过多少毫秒,不需要自己处理时间
2. 当TCP发送端初始化的时候,将会有一个`retransmission timeout (RTO)`,这就是重传时间,重传时间是变化的,但是最初的超时时间都是一样的,需要调用`_initial retransmission timeout`
3. **实现一个重传定时器:** 当重传时间到达时发出警告,超过重传时间关闭警告,只可以依赖`tick`方法,不可以根据现实时间
4. 每次一个数据报发送的时候,如果定时器没有开启,就要开启定时器
5. 当确认所有数据时，停止重传计时器
6. 如果调用`tick`后发现定时器过期:
   - 重传最早的未收到`ack`的包
   - 如果`windows size`大小不为零:
     - 你需要跟踪连续重传的数量,因为TCP连接需要据此判断连接是否出现问题,是否需要中断
     - 将重传定时器时长加倍,这叫做`exponential backoff`指数补偿,随着重传次数的增加，补偿的程度也会指数增长
   - 在超过重传时间后重置重传定时器并开启
7. 当成功收到数据
   - 设置重传时间为初始值
   - 如果发送重传数据,就需要重启定时器
   - 将连续重传记录的数量置零

## 实现TCP发送端 Implementing the TCP sender

### 思路

1. `void fill_window()`
   - TCP发送端被要求尽可能的读取`ByteStream`中的数据,并且形成数据报 ( 未发送 ) 
   - 确保每次发送的数据报的量正好等于TCP接收端窗口的大小
   - 使用`TCPSegment::length in sequence space()`得到`seq`,不要忘了SYN , FIN 也占用一个序列号,所以也占用窗口空间
2. `void ack_received(const WrappingInt32 ackno, const uint16 t window size)`
   - 得到接收端TCP窗口的左沿和大小 , `ackno`理应大于所有已发送数据报的`seq`
3. `void tick( const size t ms since last tick )`
   - **表示自从上一次发送后经过多少毫秒了.不需要我们来调用,参数的意义是距离上次调用经过了多少时间,这个不需要我们操心,我们需要实现每次调用`tick()`的时候函数做什么**
4. `void send empty segment()`
   - 生成空的数据报,只发一个`ack`

![image-20220124211130547](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201242111626.png)

### 思路总结

1. `void tick( const size t ms since last tick )`
   - **表示自从上一次发送后经过多少毫秒了.不需要我们来调用,参数的意义是距离上次调用经过了多少时间,这个不需要我们操心,我们需要实现每次调用`tick()`的时候函数做什么**
2. 我,们需要存储已发送但未被确认的报文段,进行累计确认,如果超时,只需要重传最早的报文段即可
3. `TCPReceiver` 调用 `unwrap` 时的 `checkpoint` 是上一个接收到的报文段的 `absolute_seqno`，`TCPSender` 调用` unwrap `时的 `checkpoint` 是 `_next_seqno`。
4. `retransmission timeout(RTO)`，具体实现是`RFC6298`的简化版
   - 重传连续的之后double ；收到ackno后重置到`_initial_RTO`
   - 可参考[RFC 6298](https://datatracker.ietf.org/doc/rfc6298/?include_text=1)第5小节实现_timer

### 加入成员变量

```C++
private:
    bool _syn_sent = false;
    bool _fin_sent = false;
    uint64_t _bytes_in_flight = 0;  // Number of bytes in flight 就是未被确认的字节数
    uint16_t _receiver_window_size = 0;  //接收方的滑动窗口
    uint16_t _receiver_free_space = 0; //接收方的剩余空间
    uint16_t _consecutive_retransmissions = 0; //
    unsigned int _time_elapsed = 0;
    bool _timer_running = false;                    
    std::queue<TCPSegment> _segments_outstanding{};  // the segment has been sent
    void send_segment(TCPSegment &seg);              // send segment
    bool _ack_valid(uint64_t abs_ackno);
    //! our initial sequence number, the number for our SYN.
    WrappingInt32 _isn;
    //! outbound queue of segments that the TCPSender wants sent
    std::queue<TCPSegment> _segments_out{};
    //! retransmission timer for the connection
    unsigned int _initial_retransmission_timeout;
    //! outgoing stream of bytes that have not yet been sent
    ByteStream _stream;
    //! the (absolute) sequence number for the next byte to be sent
    uint64_t _next_seqno{0};
    unsigned int _rto = 0;
public:
```

### 函数实现

#### `fill_window()` 实现

- 如果syn未发送,发送并返回
- 如果syn未应答,返回
- 如果fin已发送,返回
- 如果 _receiver_window_size 不为 0
  - 当 receiver_free_space 不为 0,尽可能地填充 payload
  - 如果 _stream 已经 EOF，且 _receiver_free_space 仍不为 0，填上 FIN（fin 也会占用 _receiver_free_space）
  - 如果 _receiver_free_space 还不为 0，且 _stream 还有内容，回到步骤 1 继续填充
- 如果 _receiver_window_size 为 0，则需要发送零窗口探测报文
  - 如果 _receiver_free_space 为 0
    - 如果 _stream 已经 EOF，发送仅携带 FIN 的报文
    - 如果 _stream 还有内容，发送仅携带一位数据的报文

### `send_segment()`实现

- 发送`seq`,不要忘记`isn`
- 记录接收方窗口大小,如果`syn true`,减去发送包的大小
- 如果计时器未开启,开启计时器

**其他的都很好理解,看一下代码就能懂**

**总体来说,有点小复杂,总有一些地方不明了,最后借鉴了一位博主的实现成功完成**

```C++
#include "tcp_sender.hh"

#include "tcp_config.hh"

#include <random>
#include <algorithm>
// Dummy implementation of a TCP sender

// For Lab 3, please replace with a real implementation that passes the
// automated checks run by `make check_lab3`.

template <typename... Targs>
void DUMMY_CODE(Targs &&.../* unused */) {}

using namespace std;

//! \param[in] capacity the capacity of the outgoing byte stream
//! \param[in] retx_timeout the initial amount of time to wait before retransmitting the oldest outstanding segment
//! \param[in] fixed_isn the Initial Sequence Number to use, if set (otherwise uses a random ISN)
TCPSender::TCPSender(const size_t capacity, const uint16_t retx_timeout, const std::optional<WrappingInt32> fixed_isn)
    : _isn(fixed_isn.value_or(WrappingInt32{random_device()()}))
    , _initial_retransmission_timeout{retx_timeout}
    , _stream(capacity)
    , _rto{retx_timeout} {}
uint64_t TCPSender::bytes_in_flight() const { return _bytes_in_flight; }

void TCPSender::fill_window() {
    if (!_syn_sent) {
        _syn_sent = true;
        TCPSegment seg;
        seg.header().syn = true;
        send_segment(seg);
        return;
    }
    if (!_segments_outstanding.empty() && _segments_outstanding.front().header().syn)
        return;
    if (!_stream.buffer_size() && !_stream.eof())
        return;
    if (_fin_sent)
        return;

    if (_receiver_window_size) {
        while (_receiver_free_space) {
            TCPSegment seg;
            size_t payload_size = min({_stream.buffer_size(),
                                       static_cast<size_t>(_receiver_free_space),
                                       static_cast<size_t>(TCPConfig::MAX_PAYLOAD_SIZE)});
            seg.payload() = _stream.read(payload_size);
            if (_stream.eof() && static_cast<size_t>(_receiver_free_space) > payload_size) {
                seg.header().fin = true;
                _fin_sent = true;
            }
            send_segment(seg);
            if (_stream.buffer_empty())
                break;
        }
    } else if (_receiver_free_space == 0) {
        // The zero-window-detect-segment should only be sent once (retransmition excute by tick function).
        // Before it is sent, _receiver_free_space is zero. Then it will be -1.
        TCPSegment seg;
        if (_stream.eof()) {
            seg.header().fin = true;
            _fin_sent = true;
            send_segment(seg);
        } else if (!_stream.buffer_empty()) {
            seg.payload() = _stream.read(1);
            send_segment(seg);
        }
    }
}

void TCPSender::send_segment(TCPSegment &seg) {
    seg.header().seqno = wrap(_next_seqno, _isn);
    _next_seqno += seg.length_in_sequence_space();
    _bytes_in_flight += seg.length_in_sequence_space();
    if (_syn_sent)
        _receiver_free_space -= seg.length_in_sequence_space();
    _segments_out.push(seg);
    _segments_outstanding.push(seg);
    if (!_timer_running) {
        _timer_running = true;
        _time_elapsed = 0;
    }
}
// See test code send_window.cc line 113 why the commented code is wrong.
//! \param ackno The remote receiver's ackno (acknowledgment number)
//! \param window_size The remote receiver's advertised window size
void TCPSender::ack_received(const WrappingInt32 ackno, const uint16_t window_size) {
    uint64_t abs_ackno = unwrap(ackno, _isn, _next_seqno);
    if (!_ack_valid(abs_ackno)) {
        // cout << "invalid ackno!\n";
        return;
    }
    _receiver_window_size = window_size;
    _receiver_free_space = window_size;
    while (!_segments_outstanding.empty()) {
        TCPSegment seg = _segments_outstanding.front();
        if (unwrap(seg.header().seqno, _isn, _next_seqno) + seg.length_in_sequence_space() <= abs_ackno) {
            _bytes_in_flight -= seg.length_in_sequence_space();
            _segments_outstanding.pop();
            // Do not do the following operations outside while loop.
            // Because if the ack is not corresponding to any segment in the segment_outstanding,
            // we should not restart the timer.
            _time_elapsed = 0;
            _rto = _initial_retransmission_timeout;
            _consecutive_retransmissions = 0;
        } else {
            break;
        }
    }
    if (!_segments_outstanding.empty()) {
        _receiver_free_space = static_cast<uint16_t>(
            abs_ackno + static_cast<uint64_t>(window_size) -
            unwrap(_segments_outstanding.front().header().seqno, _isn, _next_seqno) - _bytes_in_flight);
    }

    if (!_bytes_in_flight)
        _timer_running = false;
    // Note that test code will call it again.
    fill_window();
}

// See test code send_window.cc line 113 why the commented code is wrong.
bool TCPSender::_ack_valid(uint64_t abs_ackno) {
    return abs_ackno <= _next_seqno &&
           //  abs_ackno >= unwrap(_segments_outstanding.front().header().seqno, _isn, _next_seqno) +
           //          _segments_outstanding.front().length_in_sequence_space();
           abs_ackno >= unwrap(_segments_outstanding.front().header().seqno, _isn, _next_seqno);
}
//! \param[in] ms_since_last_tick the number of milliseconds since the last call to this method
void TCPSender::tick(const size_t ms_since_last_tick) {
    if (!_timer_running)
        return;
    _time_elapsed += ms_since_last_tick;
    if (_time_elapsed >= _rto) {
        _segments_out.push(_segments_outstanding.front());
        if (_receiver_window_size || _segments_outstanding.front().header().syn) {
            ++_consecutive_retransmissions;
            _rto <<= 1;
        }
        _time_elapsed = 0;
    }
}

unsigned int TCPSender::consecutive_retransmissions() const { return _consecutive_retransmissions; }

void TCPSender::send_empty_segment() {
    TCPSegment seg;
    seg.header().seqno = wrap(_next_seqno, _isn);
    _segments_out.push(seg);
}

```

