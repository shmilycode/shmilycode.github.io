# 简介

本文主要介绍一些在 `WebRTC` 中用到的 `RTCP` 包。

## FCI

定义在 [RFC4585]中。

全称为 `Feedback Control Information`。

所有的 `FB` 信息必须使用一个包含了如下格式的共同头部：

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |V=2|P|   FMT   |       PT      |          length               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                  SSRC of packet sender                        |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                  SSRC of media source                         |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    :            Feedback Control Information (FCI)                 :
    :                                                               :

           Figure 1: Common Packet Format for Feedback Messages

负荷类型（`Payload Types`）`PT` 的常用取值有：

Value | Abbrev | Name | Reference
----- | ------ | ---- | ---------
200	| SR | sender | report | [RFC3550]
201	| RR | receiver | report | [RFC3550]
202	| SDES | source | description | [RFC3550]
203	| BYE | goodbye | [RFC3550]
204	| APP	| application-defined	| [RFC3550]
205	| RTPFB	| Generic | RTP | Feedback | [RFC4585]
206	| PSFB | Payload-specific | [RFC4585]

`RTPFB` 负荷类型的 `FMT` 取值有：

Value | Name | Long Name | Reference
----- | ---- | --------- | ---------
1	| Generic NACK | Generic negative acknowledgement	[RFC4585]
2	| | Reserved | [RFC5104]
3	| TMMBR	| Temporary Maximum Media Stream Bit Rate Request	| [RFC5104]
4	| TMMBN	| Temporary Maximum Media Stream Bit Rate Notification | [RFC5104]
5	| RTCP-SR-REQ	| RTCP Rapid Resynchronisation Request | [RFC6051]
6	| RAMS | Rapid Acquisition of Multicast Sessions | [RFC6285]
7	| TLLEI	| Transport-Layer Third-Party Loss Early Indication	| [RFC6642]
8	| RTCP-ECN-FB	| RTCP ECN Feedback	| [RFC6679]
9	| PAUSE-RESUME | Media Pause/Resume	| [RFC7728]
10 | DBI | Delay Budget Information (DBI)	| [3GPP TS 26.114 v16.3.0][Ozgur_Oyman]
11-30	| | Unassigned	|
31 | Extension | Reserved for future extensions | [RFC4585]

` PSFB` 负荷类型的 `FMT` 取值有：

Value | Name | Long Name | Reference
----- | ---- | --------- | ---------
1	| PLI	| Picture Loss Indication	[RFC4585]
2	| SLI	| Slice Loss Indication	[RFC4585]
3	| RPSI | Reference Picture Selection Indication	[RFC4585]
4	| FIR	| Full Intra Request Command	[RFC5104]
5	| TSTR | Temporal-Spatial Trade-off Request	[RFC5104]
6	| TSTN | Temporal-Spatial Trade-off Notification	[RFC5104]
7	| VBCM | Video Back Channel Message	[RFC5104]
8	| PSLEI | Payload-Specific Third-Party Loss Early Indication	[RFC6642]
9	| ROI	| Video region-of-interest (ROI)	[3GPP TS 26.114 v16.3.0][Ozgur_Oyman]
10 | LRR | Layer Refresh Request Command	[RFC-ietf-avtext-lrr-07]
11-14	| | Unassigned |
15 | AFB | Application Layer Feedback	| [RFC4585]
16-30	| | Unassigned |
31 | Extension | Reserved for future extensions	| [RFC4585]


更多信息可以参考 [rtp-parameters]。

## TMMBR 和 TMMBN

定义在 [RFC5104] 中。

全称为 `Temporary Maximum Media Stream Bit Rate Request and Notification`，属于 `RTCP feedback` 信息类型。

`receiver`，`translator` 和 `mixer` 使用 `TMMBR` 来请求发送端将媒体流的最大比特率限制为 `TMMBR` 提供的值或以下。`TMMBN` 包含媒体发送端当前所知道的 **最大限制子集**，它是发送端接收到的 `TMMBR` 定义的限制的子集，以让参与者抑制一些会进一步限制发送方比特率的 `TMMBR` 的发送。`TMMBR / TMMBN` 消息的主要应用于有 `MCU` 或 `mixer`的情况下，既对应于 `Topo-Translator` 或 `Topo-Mixer`，也对应于 `Topo-Point-to-Point`。

`TMMBR` 包的组成首先是 `FCI` 公共头，其中的包类型字段 `PT=PTPFB` 和字段 `FMT=3`。一个 `FCI` 包含了一个或多个如下格式的 `TMMBR FCI` 入口：

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                              SSRC                             |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | MxTBR Exp |  MxTBR Mantissa                 |Measured Overhead|
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

         Figure 2 - Syntax of an FCI Entry in the TMMBR Message

`TMMBN` 包的组成首先是 `FCI` 公共头，其中的包类型字段 `PT=PTPFB` and `FMT=4`。一个 `FCI` 由0个，一个或更多的以下语法组成：

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                              SSRC                             |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | MxTBR Exp |  MxTBR Mantissa                 |Measured Overhead|
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

         Figure 3 - Syntax of an FCI Entry in the TMMBN Message

## RTCP-SR-REQ

定义在 [RFC6051]中。

全称为 `Rapid Resynchronisation Request`, 一种新的 `RTCP feedback` 信息类型。

字段 `FMT=5`，长度必须为2。如何当前使用的是 `RTP/AVPF profile` ,这种信息可能是由发送端发送，用于指示他当前无法同步媒体流，需要媒体源尽快的发送一个 `RTCP SR` 包。所以当接收到这个的一条信息后，媒体源应该产生一个 `RTCP SR` 包。

定义格式如下：

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |V=2|P|   FMT   | PT=RTPFB=205  |          length               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                  SSRC of packet sender                        |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                  SSRC of media source                         |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    :            Feedback Control Information (FCI)                 :
    :                                                               :

          Figure 4: RTP/AVPF Transport Layer Feedback Message


## NACK

定义在 [RFC4585]中。

全称为 `Negative acknowledgements`, 属于 `RTCP feedback` 信息类型。

通用 `NACK` 信息由 `PT=RTPFB` 和 `FMT=1` 标识，一个 `FCI` 域必须包含至少一个或多个通用 `NACK`。它被用于指示一个或多个 `RTP` 丢包，丢包由一个包标识符和一个位掩码表示。

定义格式如下：

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |            PID                |             BLP               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

                Figure 4: Syntax for the Generic NACK message

## PLI

定义在 [RFC4585]中。

全称为 `Picture Loss Indication`, 属于 `RTCP feedback` 信息类型。

`PLI feedback`信息由 `PT=PSFB` 和 `FMT=1`  标识，一个 `FCI` 域只能包含一个 `PLI`。视频解码器通过 `PLI` 告知编码器有关一个或多个图片的未定义数量的编码视频数据丢失。当与基于图片间预测的任何视频编码方案结合使用时，接收 `PLI` 的编码器将意识到预测链可能会中断。 发送者可以通过发送一个关键图片来实现重新同步，从而对 `PLI` 做出反应（使该消息实际上类似于 `FIR` 消息）； 但是，发送方必须考虑第7节中概述的拥塞控制，这可能会限制其发送关键帧的能力。

`PLI` 的格式不要求有其他参数，因此 `length` 字段必须为2，而且不包含任何 `Feedback Control Information`。

格式如下：

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |V=2|P|   FMT   |       PT      |          length               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                  SSRC of packet sender                        |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                  SSRC of media source                         |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    :            Feedback Control Information (FCI)                 :
    :                                                               :

## SLI

定义在 [RFC4585]中。

全称为 `Slice Loss Indication`, 属于 `RTCP feedback` 信息类型。

`SLI feedback` 信息由 `PT=PSFB` 和 `FMT=2` 标识，一个 `FCI` 域必须包含至少一个或者多个 `SLI`。解码器可以通知编码器它已按扫描顺序检测到一个或几个连续宏块的丢失或损坏。 此 `FB` 消息绝不能用于具有非均匀，动态可变宏块大小的视频编解码器，例如部分情况下的H.263，因为在这种情况下，编码器不能始终标识出损坏的空间区域。

格式如下：

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |            First        |        Number           | PictureID |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

             Figure 6: Syntax of the Slice Loss Indication (SLI)

## FIR

定义在 [RFC5104]中。

全称为 `Full Intra Request`, 属于 `RTCP feedback` 信息类型。

`FIR` 信息由 `PT=PSFB` 和 `FMT=4` 标识。全帧内请求的 `FCI` 由一个或多个 `FCI` 条目组成。`FIR` 反馈消息的长度必须设置为 `2 + 2 * N` ，其中 `N` 为 `FCI` 条目的数量。

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                              SSRC                             |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | Seq nr.       |    Reserved                                   |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

          Figure 4 - Syntax of an FCI Entry in the FIR Message

## APP

定义在 [RFC3550]中。

全称为 `Application-Defined RTCP Packet`。

`APP` 数据包被用于开发新的应用程序和新功能时的实验用途，而无需注册数据包类型值。名称无法识别的 `APP` 数据包应被忽略。 经过测试并且有理由更广泛使用之后，建议重新定义每个APP数据包，而不包含子类型和名称字段，并使用 `RTCP` 数据包类型向 `IANA` 注册。

格式如下：

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |V=2|P| subtype |   PT=APP=204  |             length            |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                           SSRC/CSRC                           |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                          name (ASCII)                         |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                   application-dependent data                ...
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

## REMB

还未被 RFC 收录，可参考 [draft-alvestrand-rmcat-remb]。

全称 `Receiver Estimated Maximum Bitrate`。

`LNTF` 信息由 `FMT=15`，`PT=PSFB` 和 `UID=REMB` 标识。该反馈消息用于在同一RTP会话上，向多个媒体流的发送方通知比特率, 该值为到达该RTP会话的接收方的路径上的总估计可用比特率。如果是符合规范的REMB包，媒体发送方会修改发送的总比特率，以使比特率等于或低于些消息中给出的值。新的比特率限制应尽快被应用，发送者可以根据自己的限制和估计自由应用其他带宽限制。

     0                   1                   2                   3               
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |V=2|P| FMT=15  |   PT=206      |             length            |                               
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                  SSRC of packet sender                        |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                  SSRC of media source                         |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |  Unique identifier 'R' 'E' 'M' 'B'                            |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |  Num SSRC     | BR Exp    |  BR Mantissa                      |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |   SSRC feedback                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |  ...                                                          |

## LNTF (Loss Notification)

在 `webrtc` 中实现于 `loss_notification.cc`中。

`LNTF` 信息由 `FMT=15`，`PT=PSFB` 和 `UID=LNTF` 标识。

从名字和包结构上来看，应该是接收端或解码端向发送端或编码端通告丢包信息的。

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |V=2|P| FMT=15  |   PT=206      |             length            |
    +=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+
    |                  SSRC of packet sender                        |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                  SSRC of media source                         |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |  Unique identifier 'L' 'N' 'T' 'F'                            |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | Last Decoded Sequence Number  | Last Received SeqNum Delta  |D|
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

## GOOG 

为 `APP` ，在 `webrtc` 中实现于 `remote_estimate.cc` 中。

命名为 `RemoteEstimate`

`GOOG` 信息由 `subtype=13`，`PT=APP` 和 `name=goog` 标识。

接收端通过 `RemoteEstimate` 包提供关于当前连接的网络统计信息。

格式为：

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |V=2|P|    13   |   PT=APP=204  |             length            |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                           SSRC/CSRC                           |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                       name 'g' 'o' 'o' 'g'                    |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |capacity_lower |capacity_upper | ...                    
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

## XR

定义在 [RFC3611]中。

全称为 `Application-Defined RTCP Packet`。

包含7种块格式，其中三种包对包块类型：

- Loss RLE Report Block: 有关RTP数据包丢失和接收的报告的 `run length` 编码。

- Duplicate RLE Report Block: 与收到的RTP数据包副本有关的报告的 `run length` 编码。

- Packet Receipt Times Report Block: RTP数据包的接收时间戳列表

还有两种参考时间相关的块类型: 

- Receiver Reference Time Report Block: 接收器端挂钟时间戳记。这些与下面提到的 `DLRR` 报告块一起，使非发送者可以计算往返时间。

- DLRR Report Block: 自上一个 `Receiver Reference Time Report Block` 以来的延迟。一个R接收到 `Receiver Reference Time Report Block` 的发送方可以用 `DLRR` 报告模块进行响应，其方式与为 `RTCP` 定义的机制相同，一个RTP数据接收者收接收到发送方的 `NTP` 时间戳后，可以通过填写 `RTCP reception report block` 的 `DLSR` 字段进行响应。

最后是两种摘要指标块类型：

- Statistics Summary Report Block: RTP数据包序列号，丢失，重复，抖动和TTL或跳数限制值的统计信息。

- VoIP Metrics Report Block: 监视IP语音（VoIP）呼叫的指标。

XR 头部格式如下：

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |V=2|P|reserved |   PT=XR=207   |             length            |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                              SSRC                             |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    :                         report blocks                         :
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

具体块类型不做介绍，太多了。

## transfport feedback

定义在[draft-holmer-rmcat-transport-wide-cc-extensions] 中，在 `webrtc` 中实现于transport_feedback.cc，也就是文章中提到的 `Transport-CC feedback`，具体内容请参考 另一篇文章[transport-wide-cc-extension]。

格式如下：

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

## SR 和 RR

定义在 [RFC3550]中。

全称为 `Sender Report RTCP Packet` 和 `Receiver Report RTCP Packet`。

RTP 接收者利用 RTCP 报告包提供接收质量反馈。根据接收者是否同时还是发送者，RTCP
包采取两种不同的形式。发送者报告(SR)和接收者报告(RR)格式中唯一的不同，除包类型码之
外，在于发送者报告包括 20 字节的发送者信息

SR 包和 RR 包都包括零到多个接收报告块，报告块中的信息包括丢包率，累计包丢失数，接收到的扩展的最高序列号，到达间隔抖动等信息。接收者报告包(RR)与发送者报告包基本相同，除了包类型域包含常数 201 和没有发送者信息的 5 个字(NTP 和 RTP 时间标志和发送者包和字节计数)。余下区域与 SR 包意义相同。若没有发送和接收据报告，在 RTCP 复合包头部加入空的 RR 包(RC=0)。

SR 格式如下：

            0                   1                   2                   3
            0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    header |V=2|P|    RC   |   PT=SR=200   |             length            |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |                         SSRC of sender                        |
           +=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+
    sender |              NTP timestamp, most significant word             |
    info   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |             NTP timestamp, least significant word             |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |                         RTP timestamp                         |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |                     sender's packet count                     |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |                      sender's octet count                     |
           +=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+
    report |                 SSRC_1 (SSRC of first source)                 |
    block  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      1    | fraction lost |       cumulative number of packets lost       |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |           extended highest sequence number received           |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |                      interarrival jitter                      |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |                         last SR (LSR)                         |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |                   delay since last SR (DLSR)                  |
           +=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+
    report |                 SSRC_2 (SSRC of second source)                |
    block  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      2    :                               ...                             :
           +=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+
           |                  profile-specific extensions                  |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+


RR 格式如下：

            0                   1                   2                   3
            0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    header |V=2|P|    RC   |   PT=RR=201   |             length            |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |                     SSRC of packet sender                     |
           +=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+
    report |                 SSRC_1 (SSRC of first source)                 |
    block  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      1    | fraction lost |       cumulative number of packets lost       |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |           extended highest sequence number received           |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |                      interarrival jitter                      |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |                         last SR (LSR)                         |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |                   delay since last SR (DLSR)                  |
           +=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+
    report |                 SSRC_2 (SSRC of second source)                |
    block  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      2    :                               ...                             :
           +=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+
           |                  profile-specific extensions                  |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

[RFC3550]: https://tools.ietf.org/html/rfc3550
[RFC3611]: https://tools.ietf.org/html/rfc3611
[RFC5104]: https://tools.ietf.org/html/rfc5104
[RFC4585]: https://tools.ietf.org/html/rfc4585
[RFC6051]: https://tools.ietf.org/html/rfc6051
[rtp-parameters]: http://www.iana.org/assignments/rtp-parameters/rtp-parameters.xhtml
[draft-alvestrand-rmcat-remb]: https://tools.ietf.org/id/draft-alvestrand-rmcat-remb-03.html
[draft-holmer-rmcat-transport-wide-cc-extensions]: https://tools.ietf.org/html/draft-holmer-rmcat-transport-wide-cc-extensions-01
[transport-wide-cc-extension]: ..\transport-wide-cc-extensions\transport-wide-cc-extension.md