# 1. 介绍

这篇文章关于一个 `RTP` 头部扩展，包含一个涵盖了整个传输周期的序列号 （`transport-wide packet sequence number`）和一个 `RTCP` 反馈信息，反馈信息包括在一条连接中收到的包的到达时间和序号。

这个扩展有如下优点：

- 拥塞控制算法更容易维护和优化，因为发送端和接收端的版本不再需要更多的同步，可以通过适当的协议实现  [I-D.ietf-rmcat-gcc], [I-D.ietf-rmcat-nada] 和 [I-D.ietf-rmcat-scream-cc]。

- 在使用哪个算法方面有更多的灵活性，因为它们大部分的逻辑都实现在发送端。例如，根据产生的速率是否受应用限制,可以应用不同的行为。

# 2. Transport-wide Sequence Number

## 2.1 语义

该 `RTP` 头部扩展被添加在传输层，包含的序号与其他通过相同连接发送的包，共用同一个计数器。

使用 `transport-wide` 序号有两个好处：

- 由于拥塞控制器不是在媒体流上运行，而是在数据包流上运行，因此它更适合于拥塞控制。

- 它允许更早的丢包检测（和恢复），因为当接收到来自流 `B` 的数据包时，就可以检测到流 `A` 的丢包，而不必等到接收到流A的下一个数据包。

## 2.2 RTP 头部扩展格式

     0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |       0xBE    |    0xDE       |           length=1            |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |  ID   | L=1   |transport-wide sequence number | zero padding  |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

每一个RTP头扩展，都带有附加到所有发送的数据包的16位序列号。 对于通过同一套接字发送的每个数据包，此序列号递增1。

## 2.3 使用该扩展的信号

在 `SDP` 中发出信号时，将会使用RTP头部扩展标准机制[RFC5285]:

    a=extmap:5 http://www.ietf.org/id/draft-holmer-rmcat-transport-wide-cc-extensions

# 3 Transport-wide RTCP Feedback Message

为了使发送端有最大的自由度，需要得到每个包的分发情况，能实现这种想法的最简单的方案，就是让接收端发一个包含了接收到的每个包的到达时间和包序号的反馈包，在这种情况下，接收端只需要记录每个包的到达时间戳（`A`）就可以了。发件端保留传输中（`in-flight`）的数据包的映射，并且在反馈到达后，查询相应的数据包的网络的在线（`on-wire`）时间戳（`S`）。从这两个时间戳中，发关端可以计算以下指标：

- Inter-packet delay variation: d(i) = A(i) - S(i) - (A(i-1) - S(i-1))

- Estimated queueing delay: q(i) = A(i) - S(i) - min{j=i-1..i-w}(A(j) - S(j))

由于发送端获取到每个发送的包的反馈，所以与由接收端确定的恒定速度相比，这种情况可以更好的评估发送突发数据包的成本 。

这种方案也有两个缺点：

- 无法区分下行链路上丢失的反馈和上行链路上丢失的数据包。

- 添加了反方向上的反馈率

从拥塞控制的角度来看，丢失的反馈消息，是通过忽略在丢失的反馈消息中，那些已被报告为丢失或接收到的数据包来处理的。 此行为类似于如何处理丢失的 `RTCP receiver report`。

建议为接收到的每个帧发送一个反馈消息，但是在上行链路带宽较低的情况下，可以不用频繁地发送它们，而是通过类似每个RTT发送一次的方式，以减少开销。

## 3.1 信息格式

该信息是一个 `payload type` 为206的 `RTCP` 信息。在 [RFC 3550] 中定义了范围，[RFC 4585]指定 `PT=206` ，`FMT=15`。

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |V=2|P|  FMT=15 |    PT=205     |           length              |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                     SSRC of packet sender                     |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                      SSRC of media source                     |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |      base sequence number     |      packet status count      |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                 reference time                | fb pkt. count |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |          packet chunk         |         packet chunk          |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    .                                                               .
    .                                                               .
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |         packet chunk          |  recv delta   |  recv delta   |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    .                                                               .
    .                                                               .
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |           recv delta          |  recv delta   | zero padding  |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

version (V): 2 bits 这个字段指示RTP版本，当前为2

padding (P): 1 bits 如果被设置，表示当前包的结束 包含额外的填充8位位组，虽然他不属于控制信息的一部分，但会被计入长度字段

feedback message type (FMT): 5 bits 指示 `FB` 信息的类型，这里必须为15

payload type (PT): 8 bits 这是RTCP数据包类型，可将数据包标识为RTCP FB消息型，此处值为205

SSRC of packet sender: 32 bits 该包的源主人的同步源标识

SSRC of media source: 32 bits 该反馈信息与之相关的媒体源的同步源标识符。TODO：作用范围是否为 `transport-wide`，是否可以选择任意媒体源SSRC？

base sequence number: 16 bits 该反馈信息中包含的第一个包的 `transport-wide` 序号，该数值并不一定在每个反馈包中都递增，在用于记录的情况下，他可能会递减。

packet status count: 16 bits 这个反馈包中包含的状态数，以 `base sequence number` 指示的包为起始。

reference time: 24 bits 有符号整数，反馈包的发送者选择的某个（未知）时基中的绝对参考时间。该值将以64ms的倍数解释。 此数据包中的第一个recv增量与参考时间有关。 即使丢失了一些反馈数据包，参考时间也可以计算出反馈之间的差异，因为它始终使用相同的时基。

feedback packet count: 8 bits 每次发送一个反馈包都会自增1，用于检测反馈包的丢失。

packet chunk: 16 bits 一个存放 `packet status chunk` 的列表，用于指示多个包的状态，以 `base sequence number` 为起始，在下面看相关的细节。

recv delta: 8 bits 对于每个在 `packet status chunks` 中的 "packet received" 状态，都会跟随一个 `receive delta block`，在下面看相关的细节。

### 3.1.1 Packet Status Symbols

一个包的状态由一个 2-bit 的符号描述：

    00 Packet not received

    01 Packet received, small delta

    10 Packet received, large or negative delta

    11 [Reserved]

有 "Packet not received" 状态的包并不一定要会解释为丢失，他们可能只是还没到达。

对于每个接收到的，与前一个接收到的包有增量的数据包，且增量在 +/- 8191.75ms内，则会有一个 `receive delta block` 被添加到反馈包里。

注意：在 `base sequence number` 减少，创建一个窗口与先前的反馈消息有重叠的情况下，则先前报告为已接收的任何数据包的状态都必须标记为 “packet not received”，这样该符号才不会包含增量。

### 3.1.2.  Packet Status Chunks

包的状态在块中被描述，与 `Loss RLE Report Block` 类似（XR RTCP中），这里有两种不同的块用于描述包的状态：

- Run length chunk

- Status vector chunk

所有的块类型长度都为16bits，块中的第一个比特用于标识这是一个 `RLE` 块还是一个 `vector` 块。

### 3.1.3 Run Length Chunk

一个游程长度块以0bit为起始位，紧跟着一个包状态和该符号的游程长度。

     0                   1
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |T| S |       Run Length        |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

chunk type (T): 1 bit 0 表示当前为游长块。

packet status symbol (S): 2 bits 在这个游程中的重复符号。

run length (L): 13 bits 无符号数，表示游程长度

例1：

     0                   1
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |0|0 0|0 0 0 0 0 1 1 0 1 1 1 0 1|
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

这是一个状态为 “packet not received” 的，长度为221的游程。

例 2:

     0                   1
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |0|1 1|0 0 0 0 0 0 0 0 1 1 0 0 0|
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

这一个是状态为“packet received, w/o recv delta”的，长度为24的游程。

### 3.1.4 Status Vector Chunk

以1 bit为起始位，然后是符号 `size bit`，最后是7个或者14个符号，依赖于 `size bit`。

     0                   1
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |T|S|       symbol list         |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

chunk type (T): 1 bit 指示当前为 `status vector chunk`

symbol size (S): 1 bit 为 0 时表示当前矢量只包含 “Packet received” (0) 和 “packet not received” （1）符号，这样我们就可以将每个符号压缩到只有一位，总共有14位。该位为1时表示当前矢量包含通常的2-bit符号，总共有7个。

symbol list: 14 bits 一个包状态符号的列表，共有7个或14个。

例1：

     0                   1
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |1|0|0 1 1 1 1 1 0 0 0 1 1 1 0 0|
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

这个块按顺序包含如下状态：

    1x "packet not received"

    5x "packet received"

    3x "packet not received"

    3x "packet received"

    2x "packet not received"

    这里的x表示增量，1x表示第一个包的状态，然后第二个5x表示 1+5=6，所以是第6个包的状态。

例 2:

     0                   1
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |1|1|0 0 1 1 0 1 0 1 0 1 0 0 0 0|
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

这个块按顺序包含如下状态：

    1x "packet not received"

    1x "packet received, w/o timestamp"

    3x "packet received"

    2x "packet not received"

### 3.1.5 Receive Delta

增量表示 `250us` 的倍数：

- 如何状态列表中添加了一个 "Packet received, small delta" 状态的符号，则一个 8-bit 的无符号接收增量会被添加到 `recv delta` 列表，代表增量的范围在 [0,63.75] ms内。

- 如何状态列表中添加了一个 "Packet received, large or negative delta" 状态的符号，则一个 16-bit 的有符号接收增量会被添加到 `recv delta` 列表，代表增量的范围在 [-8192.0, 8191.75] ms内。

- 如果增量超过最大的限制，则必须使用一个新的反馈信息，并用一个长度为24位的 `receive delta` 表示，这样就可以覆盖很大的一个范围。

注意，第一个接收增量是相对于 `base receive delta` 指示的 `reference time` 的。

`small delta`上限为63.75 ms，意味着它仅在每秒 1000/25.5=16 个数据包及以上的网络中才可用，以每个包大小为1200字节为计，相当于约150kbit/s的带宽。

0.25 ms的分辨率意味着每秒最多可以表示4000个数据包。 以1200字节/数据包为有效负载，相当于38.4 Mbit/s的有效负载带宽。


      
[RFC 3550]: https://tools.ietf.org/html/rfc3550
[RFC 4585]: https://tools.ietf.org/html/rfc4585