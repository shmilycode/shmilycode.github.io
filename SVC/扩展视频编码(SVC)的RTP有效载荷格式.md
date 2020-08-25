# RTP Payload Format for Scalable Video Coding

翻译自: https://tools.ietf.org/rfc/rfc6190.txt

# Abstract

本备忘录描述了 `Annex G of ITU-T Recommendation H.264` 所定义的可扩展视频编码(SVC)的 `RTP` 有效载荷格式，该格式在技术上与 `Amendment 3 of ISO/IEC International Standard 14496-10` 相同。  `RTP` 有效载荷格式允许在每个RTP包的有效载荷中封装一个或多个网络抽象层 `(NAL)` 单元，以及在多个 `RTP` 包中分割一个 `NAL` 单元。  此外，它还支持在一个或多个 `RTP会` 话中传输 `SVC` 流。该有效载荷格式定义了一个新的媒体子类型名称 `"H264-SVC"`，但仍然向后兼容 `RFC 6184` ，因为当基础层封装在自己的 `RTP` 流中时，必须使用 `H.264` 媒体子类型名称 `("H264")` 和 `RFC 6184` 中规定的打包方法。该有效载荷形式在视频会议、互联网视频流和高比特率娱乐质量视频等方面具有广泛的适用性。

<!-- TOC -->

- [RTP Payload Format for Scalable Video Coding](#rtp-payload-format-for-scalable-video-coding)
- [Abstract](#abstract)
- [1.  Introduction](#1-introduction)
  - [1.1.  The SVC Codec](#11-the-svc-codec)
    - [1.1.1.  Overview](#111-overview)
    - [1.1.2.  Parameter Sets](#112-parameter-sets)
    - [1.1.3.  NAL Unit Header](#113-nal-unit-header)
  - [1.2.  Overview of the Payload Format](#12-overview-of-the-payload-format)
    - [1.2.1.  Design Principles](#121-design-principles)
    - [1.2.2.  Transmission Modes and Packetization Modes](#122-transmission-modes-and-packetization-modes)
    - [1.2.3.  New Payload Structures](#123-new-payload-structures)
- [2.  Conventions](#2-conventions)
- [3.  Definitions and Abbreviations](#3-definitions-and-abbreviations)
  - [3.1.  Definitions](#31-definitions)
    - [3.1.1.  Definitions from the SVC Specification](#311-definitions-from-the-svc-specification)
    - [3.1.2.  Definitions Specific to This Memo](#312-definitions-specific-to-this-memo)
  - [3.2.  Abbreviations](#32-abbreviations)
- [4.  RTP Payload Format](#4-rtp-payload-format)
  - [4.1.  RTP Header Usage](#41-rtp-header-usage)
  - [4.2.  NAL Unit Extension and Header Usage](#42-nal-unit-extension-and-header-usage)
    - [4.2.1.  NAL Unit Extension](#421-nal-unit-extension)
    - [4.2.2.  NAL Unit Header Usage](#422-nal-unit-header-usage)
  - [4.3.  Payload Structures](#43-payload-structures)
  - [4.4.  Transmission Modes](#44-transmission-modes)
  - [4.5.  Packetization Modes](#45-packetization-modes)
    - [4.5.1.  Packetization Modes for Single-Session Transmission](#451-packetization-modes-for-single-session-transmission)
    - [4.5.2.  Packetization Modes for Multi-Session Transmission](#452-packetization-modes-for-multi-session-transmission)
  - [4.6.  Single NAL Unit Packets](#46-single-nal-unit-packets)
  - [4.7.  Aggregation Packets](#47-aggregation-packets)
    - [4.7.1.  Non-Interleaved Multi-Time Aggregation Packets (NI-MTAPs)](#471-non-interleaved-multi-time-aggregation-packets-ni-mtaps)
  - [4.8.  Fragmentation Units (FUs)](#48-fragmentation-units-fus)
  - [4.9.  Payload Content Scalability Information (PACSI) NAL Unit](#49-payload-content-scalability-information-pacsi-nal-unit)
  - [4.10.  Empty NAL unit](#410-empty-nal-unit)
  - [4.11.  Decoding Order Number (DON)](#411-decoding-order-number-don)
    - [4.11.1.  Cross-Session DON (CS-DON) for Multi-Session Transmission](#4111-cross-session-don-cs-don-for-multi-session-transmission)
- [5.  Packetization Rules](#5-packetization-rules)
  - [5.1.  Packetization Rules for Single-Session Transmission](#51-packetization-rules-for-single-session-transmission)
  - [5.2.  Packetization Rules for Multi-Session Transmission](#52-packetization-rules-for-multi-session-transmission)
    - [5.2.1.  NI-T/NI-TC Packetization Rules](#521-ni-tni-tc-packetization-rules)
    - [5.2.2.  NI-C/NI-TC Packetization Rules](#522-ni-cni-tc-packetization-rules)
    - [5.2.3.  I-C Packetization Rules](#523-i-c-packetization-rules)
    - [5.2.4.  Packetization Rules for Non-VCL NAL Units](#524-packetization-rules-for-non-vcl-nal-units)
    - [5.2.5.  Packetization Rules for Prefix NAL Units](#525-packetization-rules-for-prefix-nal-units)
- [6.  De-Packetization Process](#6-de-packetization-process)
  - [6.1.  De-Packetization Process for Single-Session Transmission](#61-de-packetization-process-for-single-session-transmission)
  - [6.2.  De-Packetization Process for Multi-Session Transmission](#62-de-packetization-process-for-multi-session-transmission)
    - [6.2.1.  Decoding Order Recovery for the NI-T and NI-TC Modes](#621-decoding-order-recovery-for-the-ni-t-and-ni-tc-modes)
      - [6.2.1.1.  Informative Algorithm for NI-T Decoding Order Recovery within an Access Unit](#6211-informative-algorithm-for-ni-t-decoding-order-recovery-within-an-access-unit)
    - [6.2.2.  Decoding Order Recovery for the NI-C, NI-TC, and I-C Modes](#622-decoding-order-recovery-for-the-ni-c-ni-tc-and-i-c-modes)
- [7.  Payload Format Parameters](#7-payload-format-parameters)
- [8.  Security Considerations](#8-security-considerations)
- [9.  Congestion Control](#9-congestion-control)
- [10.  IANA Considerations](#10-iana-considerations)
- [11.  Informative Appendix: Application Examples](#11-informative-appendix-application-examples)
  - [11.1.  Introduction](#111-introduction)
  - [11.2.  Layered Multicast](#112-layered-multicast)
  - [11.3.  Streaming](#113-streaming)
  - [11.4.  Video conferencing (Unicast to MANE, Unicast to Endpoints)](#114-video-conferencing-unicast-to-mane-unicast-to-endpoints)
  - [11.5.  Mobile TV (Multicast to MANE, Unicast to Endpoint)](#115-mobile-tv-multicast-to-mane-unicast-to-endpoint)
- [13.  References](#13-references)
  - [13.1.  Normative References](#131-normative-references)

<!-- /TOC -->

# 1.  Introduction

本备忘录为 `H.264/AVC` 视频编码标准的可扩展视频编码 `(SVC)` 扩展，规定了 `RTP[RFC3550]` 有效载荷格式。`SVC` 在 `ISO/IEC 14496 Part 10 [ISOIEC14496-10]` 的修正案3中规定，并等效地在 `ITU-T Rec.H.264[H.264]` 的附件 `G` 中规定。在本备忘录中，除非另有明确说明，`"H.264/AVC"` 是指 `[H.264]` 的规范，不包括附件 `G`。

`SVC` 涵盖了 `H.264/AVC` 的整个应用范围，从低比特率的移动应用，到高清晰度电视 `(HDTV)` 广播，甚至是需要近乎无损编码和每秒数百兆比特的数字电影。`SVC` 在 `H.264/AVC` 中增加的可扩展性功能，使系统能够在不处理或最少处理的情况下，使信号适应不同的系统条件，这种适应性既与潜在的异质接收器的能力有关（在屏幕分辨率、处理速度等方面存在差异），也与不同的或会随时间变化的网络条件有关，适应性可以在源端、目的端或中间媒体感知网元 `(MANEs)` 进行。本备忘录中指定的有效载荷格式暴露了这些系统级功能，因此系统设计者可以直接利用这些功能。

* 信息性说明：由于 `SVC` 流在设计上包含了一个符合 `H.264/AVC` 标准的子流，因此 `MANE` 可以很容易地过滤该流，从而去除所有 `SVC` 特定信息。实际上，本备忘录定义了一个媒体类型参数( `sprop-avc-ready`，第7.2节) ，该参数表示是否可以将流转换为符合 `[RFC6184]` 的流，方法是过滤 `RTP` 数据包，并重写 `RTP` 控制协议 `(RTCP)` 以匹配 `[RFC3550]` 第7节规定的 `RTP` 数据包流的变化。

本备忘录定义了两种基本的 `SVC` 数据传输模式，即单会话传输 `single-session transmission (SST)` 和多会话传输 `multi-session transmission（MST）`。  在 `SST` 中，一个 `RTP` 会话用于传输由 `SVC` 比特流组成的所有可扩展性层；在 `MST` 中，可扩展性层在不同的 `RTP` 会话上传输。在 `SST` 中，打包是对 `[RFC6184]` 的直接扩展。对于 `MST` ，本备忘录中定义了四种不同的模式，它们的区别在于是否允许交织 `(interleaving)`，即以与解码顺序不同的顺序传输网络抽象层 `(NAL)` 单元，以及用于实现会话间 `NAL` 单元解码顺序恢复的技术。解码顺序恢复是通过使用会话间时间戳对齐 `[RFC3550]` 或跨会话解码顺序号 `(CS-DONs)` 来实现的。  其中一种 `MST` 模式支持两种解码顺序恢复技术，因此接收者可以选择自己喜欢的技术。更多细节可参见 1.2.2 节。

本备忘录进一步定义了三种新的 `NAL` 单元类型。第一种类型是载荷内容可扩展性信息 `payload content scalability information (PACSI) NAL` 单元，用于提供 `RTP` 数据包中包含的数据的可扩展性信息摘要，以及辅助数据(如 `CS-DON` 值)。第二和第三种新的 `NALU` 类型是空 `NAL` 单元和非交错多时间聚合包 `non-interleaved multi-time aggregation packet (NI-MTAP) NAL` 单元。  空 `NAL` 单元用于确保 `MST` 中解码顺序恢复所需的会话间时间戳对齐。`NI-MTAP` 作为一种新的有效载荷结构，允许不同时间实例的 `NAL` 单元按解码顺序分组。关于新的数据包结构的更多细节可以在 1.2.3 节中找到。

该备忘录还定义了可以通过 `RTP` 传输的 `SVC` 信令，包括一个新的媒体子类型名称 `（H264-SVC）`。

本节剩余部分将对 `SVC` 编解码器和有效载荷进行非规范性概述。

## 1.1.  The SVC Codec

### 1.1.1.  Overview

`SVC` 定义了一种编码的视频表现形式，其中给定的比特流在不同的保真度水平上对源材料进行表现(因此称为 "可扩展")。  可扩展的视频编码比特流，或可扩展的比特流，是以金字塔的方式构造的：编码过程中创建的比特流组件，可以改善层次较低组件的保真度。

`SVC` 提供的保真度维度是空间（图片大小）、质量（或信噪比 （`SNR`） ）和时间（`pictures per second`）。与给定的空间、质量和时间保真度水平关联的比特流组件，是通过比特流中相应的参数来识别的：`dependency_id` 、`quality_id` 和 `temporal_id` （另见1.1.3节）。保真度标识符e采用整数值，数值越高，表示组件在层次结构中越高。值得注意的是，`SVC` 在编码器如何选择构建各组件之间的依赖关系方面提供了很大的灵活性。某一特定组件的解码，需要保证它所依赖的所有组件的可用性的情况下才可以完成，无论是直接的还是间接的。  一个 `SVC` 比特流的操作点由能够解码一个特定的 `dependency_id` 、`quality_id` 和 `temporal_id` 组合所需的比特流组件组成。

在本备忘录中，"层" 一词在各种情况下使用。  例如，在 "视频编码层 "和 "网络抽象层 "这两个术语中，它指的是概念组织层次。当提到比特流语法元素，如块层或宏块层时，它指的是层次化的比特流结构层次。当使用在比特流可扩展性的上下文中时，如 "AVC基础层"，它指的是包含一组特定的 `NAL` 单元的源信号的表现的保真度水平。通过提供适当的上下文来支持正确的解释。

`SVC` 保留了 `H.264/AVC` 中引入的比特流组织，具体来说，所有比特流组件都被封装在网络抽象层(`NAL`)单元中，这些单元被组织成访问单元(`AUs`)。 一个 `AU` 在时间上与一个单一的采样实例相关联。`NAL` 单元类型的一个子集对应于视频编码层(`VCL`)，并包含与源内容相关的编码图像数据。`Non-VCL` 的 `NAL` 单元携带了解码所需的辅助数据（如下面解释的参数集）或方便某些系统操作但解码过程本身不需要的数据。不同保真度下的编码图片数据是以片为单位组织的。 在一个 `AU` 内，一个操作点的编码图象由解码所需的所有编码片组成，再进一步由 `AU` 对应的时间实例的 `dependency_id`, 和` quality_id` 值特定组合。

值得注意的是，`H.264/AVC` 中已经出现了时间可扩展性的概念，因为[H.264]附件A中定义的 `profile` 已经支持这一概念。  具体来说，在 `H.264/AVC` 中，已经引入了子序列的概念，以允许通过补充增强信息(`SEI`)来选择使用时间层( `temporal-layers`)。`SVC` 通过使用 `temporal_id` 参数，与分别用于空间和质量可扩展性的 `dependency_id` 和 `quality_id` 值一起，来扩展这种方法。对于使用[H.264]附件G中定义的编码方式编码的图片数据，可以通过使用一种新类型的 `NAL` 单元来实现，即通过将编码片存放在在可扩展 `NALU (scalable extension NALU)` (类型20)中，保真度参数存放在头部。对于遵循 `H.264/AVC` 的编码图片数据，为了确保与现有的 `H.264/AVC` 解码器的兼容性，定义了另一种新类型的 `NAL` 单元，即前缀 `NAL` 单元（`prefix NAL`）(类型14)，以携带该头信息。SVC还另外指定了第三种新类型的 `NAL` 单元，即子集序列参数集 `NAL` 单元（`subset sequence parameter set NAL`）(类型15)，以包含质量和空间增强层的序列参数集信息。这三种新指定的 `NAL` 单元类型(14、15和20)都属于 `H.264/AVC` 中保留的类型，但在[H.264]附件A中指定的一个或多个 `profile` 的解码器可以忽略这些类型。

在一个 `AU` 中，与给定的 `dependency_id` 和 `quality_id` 相关联的 `VCL NAL` 单元被称为 "层表示"（`"layer representation"`）。  与 `dependency_id` 和 `quality_id` 的最低值相对应的层表示（即两者均为零），在设计上符合 `H.264/AVC` 的要求。在一个比特流中，与 `dependency_id` 和 `quality_id` 的特定值组合相关联的所有 `AU` 的 `VCL` 和相关的 `Non-VCL NAL`单元的集合，且不管 `temporal_id` 的值如何，在概念上是一个可扩展层。然而，为了向后兼容 `H.264/AVC`，重要的是要区分在给定的比特流中是否存在 `SVC` 特有的 `NAL` 单元。这对于最低保真值的 `dependency_id` 和 `quality_id` (两者均为零)尤为重要，因为相应的 `VCL` 数据是符合 `H.264/AVC` 标准的，并且可能会或不会伴随着相关的前缀 `NAL` 单位。因此，本备忘录使用术语 "`AVC`基础层" 来指定不包含 `SVC` 特定 `NAL` 单元的层，使用 "`SVC`基础层" 来指定相同，但增加了相关的 `SVC` 前缀 `NAL` 单元的层。 请注意，`SVC` 规范使用 "基础层 "一词来表示本备忘录中的 "`AVC` 基础层"。 同样，能够在一个层内区分它所包含的时间保真度组件也很重要。本备忘录使用术语 "T0" 来表示，在特定层内，包含与 `temporal_id` 等于0相关的 `NAL` 单元的子集。

`SVC` 中的 `SNR` 可扩展性有两种不同的方式。在所谓的粗粒度可扩展性(`CGS`)中，可扩展性是通过在解码一个特定比特流时排除或剔除一个完整的层来提供的，而在中粒度可扩展性(`MGS`)中，可扩展性是通过有选择地省略属于 `MGS` 的特定 `NAL` 单元来提供的。可以根据 `NALU` 头中的固定长度字段来选择要省略的 `NAL` 单元（另见1.1.3和4.2节）。

### 1.1.2.  Parameter Sets

`SVC` 保留了 `H.264/AVC` 中的参数集概念，并引入了一种新的序列参数集，称为子集序列参数集[H.264]。子集序列参数集的 `NAL` 单元类型等于 `15`，这与序列参数集的 `NAL` 单元类型值（`7`）不同。`NAL` 单元类型1～5的 `VCL NAL` 单元必须只(间接地)引用序列参数集，而 `NAL` 单元类型 `20` 的 `VCL NAL` 单元必须只(间接地)引用子集序列参数集。这些引用是间接的，因为 `VCL NAL` 单元引用图片参数集（在其片头中），而图片参数集又引用常规或子集序列参数集。子集序列参数集与序列参数集各自使用单独的标识符值空间。

在 `SVC` 中，来自不同层的编码图片数据可能使用相同或不同的序列和图片参数集。 让变量 `DQId` 等于 `dependency_id * 16 + quality_id`。 在解码过程中的任何时间瞬间，对于 `DQId` 值最高的层表示有一个活动序列参数集，对于 `DQId` 值较低的层表示有一个或多个活动分层 `SVC` 序列参数集。活动序列参数集或活动分层 `SVC` 序列参数集，在被引用的活动序列参数集或活动分层 `SVC` 序列参数集所在的可扩展层的整个编码视频序列中保持不变。 这意味着所引用的序列参数集或子集序列参数集，只能在任何层的瞬时解码刷新( `IDR` )访问单元处改变。 在解码过程中的任何时间瞬间，可以有一个活动图片参数集(用于具有最高 `DQId` 值的层表示)和一个或多个活动层图片参数集(用于具有较低 `DQId` 值的层表示)。活动图片参数集或活动分层图片参数集，在被引用的活动图片参数集或活动层图片参数集所在的层表示中保持不变，但可能从一个 `AU` 到下一个 `AU` 的过程中发生改变。

### 1.1.3.  NAL Unit Header

`SVC` 为一字节的 `H.264/AVC NALU` 头部扩展了三个额外的八位字节，用于 `14` 和 `20` 这两种类型的 `NAL` 单元。该头表示了 `NAL` 单元的类型、`NAL` 单元有效载荷中（潜在的）存在的位错误或语法错误、有关 `NAL` 单元对解码过程的相对重要性的信息、层识别信息以及下文讨论的其他字段。

`NAL` 单元头的语法和语义在[H.264]中作了规定，但为了方便起见，下文对 `NAL` 单元头的基本属性作了总结。

`NAL` 单元头的第一个字节的格式如下( `bit` 字段与一字节 `H.264/AVC NALU` 头的定义相同，但有些字段的语义以向后兼容的方式略有改变)。

         +---------------+
         |0|1|2|3|4|5|6|7|
         +-+-+-+-+-+-+-+-+
         |F|NRI|  Type   |
         +---------------+

下文简要介绍了[H.264]中规定的 `NAL` 单位八位组的组成部分的语义。除了每个字段的名称和大小外，还提供了[H.264] 中相应的语法元素名称。

F：1 bit； forbidden_zero_bit。`H.264/AVC` 将值 `1` 规定为语法错误。

NRI：2 bits；nal_ref_idc。  值为 "00" (二进制形式)表示 `NAL` 单元的内容，不会被用于重建用于未来预测的参考图片，这样的 `NAL` 单元可以被丢弃，而不会影响到同层参考图片的完整性。大于 "00" 的值表示需要对 `NAL` 单元进行解码，以保持同一层中参考图片的完整性，或者 `NAL` 单元包含参数集。

Type：5 bits；nal_unit_type。  该组件指定了[H.264]表7-1和本备忘录中后来定义的 `NAL` 单元类型，关于所有当前定义的 `NAL` 单元类型及其语义的参考，请参考[H.264]第7.4.1节。

在 `H.264/AVC` 中，`NAL` 单元类型 `14、15 和 20` 是为将来的扩展保留的。`SVC` 使用的三种 `NAL` 单元类型如下： 类型值为 `14` 的 `NALU` 为前缀NAL单元，类型值为 `15` 的 `NALU` 为子集序列参数集，类型值为 `20` 的 `NALU` 为可伸缩扩展中的编码片（见[H.264]中的7.4.1节）。  `NALU` 类型 `14` 和 `20` 表示在NAL单元头中存在三个附加八位字节，如下图所示。

            +---------------+---------------+---------------+
            |0|1|2|3|4|5|6|7|0|1|2|3|4|5|6|7|0|1|2|3|4|5|6|7|
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            |R|I|   PRID    |N| DID |  QID  | TID |U|D|O| RR|
            +---------------+---------------+---------------+

R: 1 bit；reserved_one_bit。为将来的扩展保留位。R必须等于1，R的值必须被解码器忽略。

I：1 bit；idr_flag。该组件指定层表示是否是瞬时解码刷新（`IDR`）层表示，`1` 时表示 `true`。

PRID：6 bits；priority_id。  该标志指定 `NAL` 单元的优先级标识符。`PRID` 的值越低，表示优先级越高。

N: 1 bit；no_inter_layer_pred_flag。该标志指定在解码时是否可以使用层间预测（当等于1时为 `true`）。

DID：3 bits；dependency_id。该组件表示一个层表示的层间编码依赖程度。在任何一个访问单元，具有给定 `dependency_id` 的层表示可以用于层间预测对具有较高 `dependency_id` 的层表示的编码，而不得用于层间预测对具有较低 `dependency_id` 的层表示的编码。

QID：4 bits；quality_id。该组件表示一个 `MGS` 层表示的质量等级。在任何访问单元，对于相同的 `dependency_id` 值，`quality_id` 等于 `ql` 的层表示使用 `quality_id` 等于 `ql-1` 的层表示进行层间预测。

TID:  3 bits；temporal_id。该分量表示图层表示的时间级别。`temporal_id` 与帧率相关，较低的 `temporal_id` 值对应较低的帧率。一个在给定的 `temporal_id` 的层表示通常依赖于 `temporal_id` 值较低的层表示，但它从不依赖于 `temporal_id` 值较高的层表示。

U：1 bit；use_ref_base_pic_flag。值为 `1` 表示在帧内预测过程中只参考基础图片，值为 `0` 表示在帧内预测过程中不参考基础图片。

D：1 bit；discardable_flag。值为 `1` 表示当前的 `NALU` 不用于在当前和所有后续访问单元中对 `dependency_id` 值高于当前 `NAL` 单元的NAL单元进行解码。`discardable_flag` 等于0表示需要对 `NAL` 单元进行解码，以保证具有较高 `dependency_id` 值的层的完整性。

O：1 bit；output_flag。 影响[H.264]附件C中定义的解码图像输出过程。

RR: 2 bits；reserved_three_2bits。  为将来的扩展保留位。`RR` 必须等于 "11"（二进制形式）。RR的值必须被解码器忽略。

本备忘录扩展了[H.264]附件G中的 `F、NRI、I、PRID、DID、QID、TID、U` 和 `D` 的语义，如4.2节所述。

## 1.2.  Overview of the Payload Format

与[RFC6184]类似，这种有效载荷格式只能用于在 `RTP` 上携带原始的 `NALU` 流，而不是[H.264]附件B中规定的字节流格式。

本节总结了设计原理、传输模式、分组模式以及新的有效载荷结构。这里会假设读者熟悉[RFC6184]中定义的术语和概念。

### 1.2.1.  Design Principles

这种有效载荷格式遵循了以下设计原则：

* 尽可能向后兼容[RFC6184]。

* `SVC` 基础层或 `SVC` 基础层的任何 `H.264/AVC` 兼容子集，在其自身的 `RTP` 流中传输时，必须使用[RFC6184]进行封装。这确保了这样的RTP流能够被[RFC6184]接收器理解。

* [RFC6184]中定义的媒体感知网元(`Media-aware network elements - MANEs`)是信号感知的，依赖于信令信息，具有状态。

* `MANEs` 可以聚合多个 `RTP` 流，可能来自多个 `RTP` 会话。

* `MANEs` 可以进行媒体感知流稀释(有选择地删除数据包或其中的一部分)。通过使用 `RTP` 会话中的有效载荷头信息识别层，`MANEs` 能够从传入的RTP数据包流中删除数据包或其中的一部分。这意味着需要重写输出的数据包流的 `RTP` 头，以及重写[RFC3550]第7节规定的 `RTCP` 数据包。

### 1.2.2.  Transmission Modes and Packetization Modes

该备忘录允许单会话传输（`SST`）和多会话传输（`MST`）的 `SVC` 数据打包。在 `SST` 的情况下，所有的 `SVC` 数据都在一个 `RTP` 会话中传输。  在 `MST` 的情况下，根据本备忘录中定义的 `MST` 特定分组模式，使用两个或多个 `RTP` 会话来携带 `SVC` 数据，这些模式基于[RFC6184]中定义的分组模式。在 `MST` 中，每个 `RTP` 会话与一个 `RTP` 流相关联，该 `RTP` 流可以携带一个或多个层。

基层在设计上与 `H.264/AVC` 兼容，在传输过程中，如果存在由 `SVC` 引入的，封装在与 `H.264/AVC VCL NALU` 相同的 `RTP` 数据包流中，或在不同的 `RTP` 数据包流中(当使用 `MST`时)的前缀 `NAL` 单元，则被 `H.264/AVC` 解码器忽略。为方便起见，术语 "AVC基础层" 用于指没有前缀 `NAL` 单元的基础层，而术语 "SVC基础层 "用于指有前缀 `NAL` 单元的基础层。

此外，基础层可以具有多个时间分量(即，支持不同的帧速率)。因此，`AVC` 或 `SVC` 基础层的低时态分量("T0")被用作 `SVC` 比特流层次结构的起始点。

该备忘录允许在给定的 `RTP` 流中封装以下三种层组合中的任何一种。

1. 仅有 `T0 AVC` 基础层或 `T0 SVC` 基础层；

2. 仅有一个或多个增强层；

3. `T0 SVC` 基础层，以及一个或多个增强层。

`SST` 应该用于点对点的单播应用，一般来说，当使用多个 `RTP` 会话的潜在好处不能证明增加的复杂性是合理的。当使用 `SST` 时，可以使用上述第1和第3层组合情况。当使用 `SST` 传输一个与 `H.264/AVC` 兼容的 `SVC` 基础层子集时，必须使用[RFC6184]的打包方式，从而确保与[RFC6184]接收机的兼容性。然而，当使用 `SST` 传输一个或多个 `SVC` 质量或空间增强层时，必须使用本备忘录中定义的分组。在SST中，可以使用三种[RFC6184]打包模式中的任何一种，即单 `NAL` 单元模式、非交错模式和交错模式。

当不同的接收者要求不同的可扩展比特流层时，应在多播会话中使用 `MST`。本备忘录中定义的 `SVC` 比特流的一个操作点对应于一组层，这些层共同符合[H.264]附件A或G中定义的 `profile` 之一，并在解码时以一定的保真度提供原始视频的表示。`MST` 中使用的流的数量至少应等于接收机可能要求的操作点的数量。根据不同的应用，这可能会导致每一层都在自己的 `RTP` 会话中传输，或在一个 `RTP` 会话中封装多个层。

* 信息性说明：分层组播是一个常用的术语，用来描述组播被用于传输分层或可扩展数据的应用，这些数据已被封装成一个以上的 `RTP` 会话。这种应用允许组播会话中的不同接收器接收可扩展比特流的不同操作点。分层组播以及其他应用实例将在第11.2节详细讨论。

当使用 `MST` 时，上述三种层组合中的任何一层都可以用于每个会话。  当一个 `H.264/AVC` 兼容的 `SVC` 基础层子集在自己的会话中以 `MST` 方式传输时，必须使用[RFC6184]的打包方式，这样[RFC6184]接收器可以成为 `MST` 的一部分，只接收这个会话。  对于 `MST` ，本备忘录定义了四种不同的 `MST` 专用打包模式，即基于非交错时间戳(`NI-T`)的模式、基于非交错 `CS-DON(NI-C)` 的模式、非交错组合时间戳和 `CS-DON` 模式(`NI-TC`)，以及基于交错 `CS-DON(I-C)` 的模式(详见4.5.2节)。这些模式的不同取决于是否允许 `SVC` 数据被交错，即以不同于预期解码顺序的顺序进行传输，而且它们在提供的机制上也有所不同，以便在多个 `RTP` 会话中恢复 `NAL` 单元的正确解码顺序。这四种 `MST` 模式重复使用[RFC6184]中介绍的分组模式，对每个 `RTP` 会话中的 `NAL` 单元进行分组。

正如 `MST` 封装模式的名称所暗示的那样，`NI-T、NI-C` 和 `NI-TC` 模式不允许交错传输，而 `I-C` 模式允许交错传输。在这三种非交错的 `MST` 封装模式中的任何一种模式下，实施了[RFC6184]指定的非交错模式的传统[RFC6184]接收机可以加入 `SVC` 的多会话传输，接收根据[RFC6184]封装的基础 `RTP` 会话。

### 1.2.3.  New Payload Structures

[RFC6184]规定了三种基本的有效载荷结构，即单 `NAL` 单元数据包（`single NAL unit packet`）、聚合数据包（`aggregation packet`）和分段单元（`fragmentation unit`）。根据基本的有效载荷结构，一个 `RTP` 数据包可能包含：一个不聚合其他 `NAL` 单元的 `NAL` 单元、一个聚合了一个或多个`NAL` 单元的 `NAL` 单元，或一个不聚合其他 `NAL` 单元的 `NAL` 单元分段。在[H.264]中指定类型的每个 `NAL` 单元(即，1至23，包括在内)，可以在单个 `NAL` 单元包中完整地携带，可以在一个聚合包中聚合，或者可以被碎片化并在若干分段单元包中携带。为了实现 `NAL` 单元的聚合或分段，同时确保 `RTP` 数据包的有效载荷只由 `NAL` 单元组成，[RFC6184]引入了六种新的 `NAL` 单元类型(24-29)作为有效载荷结构，这些类型是从[H.264]中未规定的 `NAL` 单元类型中选择的。

本备忘录重用了[RFC6184]中使用的所有有效载荷结构，此外，还定义了三种新类型的 `NAL` 单元：载荷内容可扩展性信息 `(payload content scalability information - PACSI) NAL` 单元、空 `NAL` 单元和非交错多时聚合包(`NI-MTAP`)(分别在4.9、4.10和4.7.1节中规定)。

`PACSI NAL` 单位可用于以下目的：

* 使 `MANEs` 能够决定是否转发、处理或丢弃聚合数据包，方法是在 `PACSI NAL` 单元中检查聚合 `NAL` 单元的可扩展性信息和其他特性，而不是检查聚合 `NAL` 单元本身，因为后者是由视频编码规范定义的。

* 借助 `PACSI NAL` 单元中包含的 `CS-DON` 信息，在 `MST` 中使用 `NI-C` 或 `NI-TC` 模式恢复正确的解码顺序。

* 提高对数据包丢失的容错性，例如，利用 `PACSI NAL` 单元中包含的以下数据或信息：重复的补充增强信息(`SEI`)消息，有关层表示的开始和结束的信息，以及最低时间子集的层表示的索引。

空 `NAL` 单元可用于在 `MST` 中使用 `NI-T` 或 `NI-TC` 模式恢复正确的解码顺序。`NI-MTAP NAL` 单元可用于聚合多个访问单元的 `NAL` 单元，但不存在交错。

# 2.  Conventions

本文档中的关键词 "MUST"、"MUST NOT"、"REQUIRED"、"SHALL"、"SHALL NOT"、"SHOULD"、"SHOULD NOT"、"RECOMMENDED"、"MAY "和 "OPTIONAL "应按照BCP 14，RFC 2119[RFC2119]中的描述进行解释。

本规范在处理 `bit` 字段时使用了设置和清除 `bit` 的概念。设置一个 `bit` 与将该 `bit` 的值定为1(`On`)是一样的，清除一个 `bit` 与将该 `bit` 的值定为0(`Off`)是一样的。

# 3.  Definitions and Abbreviations

## 3.1.  Definitions

本文件使用[H.264]的术语和定义。  为方便起见，第3.1.1节列出了从[H.264]中复制的相关定义。

当出现差异时，以[H.264]中的定义为准。  第3.1.2节给出了本备忘录特有的定义，第3.1.2节中的一些定义也存在于[RFC6184]中，根据需要略作修改后复制到这里。

### 3.1.1.  Definitions from the SVC Specification

* 访问单元。 一组NAL单元，总是正好包含一张主要的编码图片。除主要的编码图片外，一个访问单元还可以包含一个或多个冗余编码图片、一个辅助编码图片，或其他不包含编码图片的片或片数据分区的 `NAL` 单元。一个访问单元解码的结果总是一张被解码的图片。

* 基础层。比特流子集，它包含所有NAL单元，其 `nal_unit_type` 语法元素等的 `1` 或 `5` ，不包含任何 `nal_unit_type` 语法元素等于 `14`、`15` 或 `20` 的 `NAL` 单元，并符合[H.264]附件A中规定的一个或多个 `profile`。

* 基础质量层表示。 与 `quality_id` 语法元素等于 `0` 相关联的访问单元的目标依赖性表示的层表示。

* 编码视频序列。 按编码顺序由一个 `IDR` 访问单元之后是0个或更多的非 `IDR` 访问单元组成的访问单元序列，直至但不包括任何后续 `IDR` 访问单元。

* 依赖性表示。 在一个访问单元内的，视频编码层(`VCL`) `NAL` 单元的子集，通过相同的 `dependency_id` 语法元素值相关联，该元素作为 `NAL` 单元头的一部分或由相关前缀 `NAL` 单元提供。一个依赖表示由一个或多个层表示组成。

* IDR访问单元。 主要编码图像为 `IDR` 图像的访问单元。

* 层表示。 访问单元内的 `VCL NAL` 单元的一个子集，这些单元与作为 `VCL NALU` 头部的一部分，或由相关的前缀 `NAL` 单元提供的 `dependency_id` 和 `quality_id` 语法元素的相同值相关联。一个或多个层表示代表一个依赖性表示。

* 前缀 `NAL` 单元。 一个 `nal_unit_type` 等于 `14` 的 `NAL` 单元，在解码顺序中紧接在 `nal_unit_type` 等于 `1、5` 或 `12` 的 `NAL` 单元之前，在解码顺序上紧接前缀 `NAL` 单元的 `NAL` 单元被称为关联 `NAL` 单元。前缀 `NAL` 单元包含与关联 `NAL` 单元相关联的数据，这些数据被认为是关联 `NAL` 单元的一部分。

* 参考基础图片。 一张参考图片，对一个访问单元的基本质量层表示进行解码时得到的，其中`nal_ref_idc` 语法元素不等于 `0`，`store_ref_base_pic_flag` 语法元素等于 `1`，该访问单元的所有层表示都是通过基本质量层表示的层间预测得到。参考基础图片不是解码过程的输出，但参考基础图片的样本可用于解码过程中对后续图片按解码顺序进行层间预测。参考基础图片是参考基础场或参考基础帧的统称。

* 可扩展比特流。 具有以下特性的比特流：一个或多个 `不同于可扩展比特流` 的比特流子集，构成另一个符合 `SVC` 规范[H.264]的比特流。

* 目标依赖性表示。 访问单元的依赖性表示，与访问单元中所有依赖性表示中 `dependency_id` 语法元素值最大的一个相关联。

* 目标层表示。 访问单元的目标依赖性表示的层表示，该层表示与该访问单元目标依赖性表示中，所有的层表示的 `quality_id` 语法元素值最大的一个相关联。

### 3.1.2.  Definitions Specific to This Memo

锚层表示。锚层表示是这样一种层表示，即如果对应于该层的操作点的解码从包含该层表示的存取单元开始，则该层的所有以下层表示按输出顺序可以正确解码。  在[H.264]中，输出顺序被定义为解码器的解码图片缓冲区的输出顺序。由于 `H.264` 没有指定图片显示过程，所以用这个更通用的术语代替显示顺序。锚层表示是锚层表示所属层的随机访问点。然而，有些层表示，在解码顺序上晚于锚层表示，但在输出顺序上先于锚层表示，可能会参考早期的层表示进行相互预测，因此如果在锚层表示处进行随机访问，解码结果可能会出错。

`AVC` 基础层。 `SVC` 基础层的子集，其中删除了所有前缀 `NAL` 单元(类型14)。请注意，这相当于[H.264]附件G中定义的 "基础层"。

基础 `RTP` 会话。 当使用多会话传输时，携带包含 `T0 AVC` 基础层或 `T0 SVC` 基础层以及零个或多个增强层的 `RTP` 流的 `RTP` 会话。该 `RTP` 会话不依赖于任何其他 `RTP` 会话，如第7.2.3节中定义的机制所示。基础 `RTP` 会话可以携带 `NAL` 单位类型等于 `14` 和 `15` 的 `NAL` 单位。

解码顺序号(`DON`)。有效载荷结构或衍生变量中的一个字段，表示 `NAL` 单位解码顺序。`DON` 的取值范围是 `0` 到 `65535`。在达到最大值后，`DON` 的值就会环绕为 `0` 。注意，这个定义在[RFC6184]中也存在，形式完全相同。

空 `NAL` 单元。 `NAL` 单元类型等于31，子类型等于 `1` 的 `NAL` 单元。  一个空 `NAL` 单元仅由两个字节的 `NALU` 头和一个空的有效载荷组成。

增强型 `RTP` 会话。 当使用多会话传输时，指不是基本 `RTP` 会话的 `RTP` 会话。增强型 `RTP` 会话通常包含一个 `RTP` 流，该 `RTP` 流依赖于至少一个其他 `RTP` 会话，如第7.2.3节中定义的机制所示。一个相对于增强型 `RTP` 会话的更低级别 `RTP` 会话即增强型 `RTP` 会话所依赖的 `RTP`会话。接收端的最低 `RTP` 会话是不依赖于接收端接收的任何其他 `RTP` 会话的 `RTP` 会话，接收端的最高 `RTP` 会话是指没有一个 `RTP` 会话会依赖于它的 `RTP` 会话。

跨会话解码顺序号（`CS-DON`）。 一个派生变量，表示在所有携带相同 `SVC` 比特流的会话复用 `RTP` 会话中所有 `NAL` 单元的解码顺序号。

默认级别。 `profile-level-id` 参数所表示的级别，在会话描述协议(`SDP`)的 `Offer/Answer`中，级别是可以降级的，即应答可以使用默认级别，也可以使用较低的级别。 请注意，这个定义也存在于[RFC6184]中，形式略有不同。

默认 `sub-profile` 。 编码工具的子集，可以是一个 `profile` 的所有编码工具，也可以是多个 `profile` 的编码工具的共同子集，由 `profile-level-id` 参数表示。在 `SDP Offer/Answer` 中，默认 `sub-profile` 必须以不对称的方式使用，即 `Answer` 必须使用与 `Offer` 相同的 `sub-profile` 或拒绝 `Offer`。注意，这个定义也存在于[RFC6184]中，形式略有不同。

增强层。 其中至少有一个 `dependency_id` 或 `quality_id` 值大于0的层，或其中没有任何一个 `NAL` 单元与 `temporal_ide` 等于 `0` 的值有关联的层。 使用与增强层相关联的最大 `temporal_id、dependency_id` 和 `quality_id` 值构建的操作点可以符合或不符合[H.264]附件A中规定的一个或多个 `profile`。

`H.264/AVC` 兼容。比特流子集符合[H.264]附件A中规定的一个或多个 `profile` 的属性。

`intra` 层表示。  一种层表示，它只包含使用帧内预测的片，因此在同一层的解码顺序中不引用任何早期的层表示。注意，在 `SVC` 中，帧内预测包括层内帧内预测以及层间帧内预测。

层。一个比特流子集，其中所有类型 `1、5、12、14` 或 `20` 的 `NAL` 单元具有相同的 `dependency_id` 和 `quality_id` 值，可以直接通过它们的 `NAL` 单元头（类型 `14` 或 `20` 的 `NAL` 单元）或通过关联到前缀（类型`14`） `NAL`单元（对于类型 `1、5` 或 `12` 的 `NAL` 单元）获取到。 一个层可以包含与多个 `temporal_id` 值关联的 `NAL` 单元。

媒体感知网元(`MANEs`)：一个网络元素，如中间盒或应用层网关，它能够解析 `RTP` 有效载荷头或 `RTP` 有效载荷的某些方面并对其内容作出反应。请注意，这个定义也存在于[RFC6184]中，形式完全相同。

* 信息性说明：`MANE` 的概念超出了普通路由器或网关的范围，因为 `MANE` 必须了解信令(如了解媒体流的有效载荷类型映射)，而且在与安全实时传输协议(`SRTP`)合作时必须得到信任。使用 `MANEs` 的优点是，它们允许根据媒体编码的需要丢弃数据包。例如，如果在某条链路上出现拥塞，`MANE` 必须丢弃数据包，它可以识别并删除那些就算去掉也会对用户体验产生最小不利影响的数据包。在丢弃数据包后，`MANE` 必须按照[RFC3550]第7节的规定重写 `RTCP` 数据包，以匹配 `RTP` 数据包流的变化。

多会话传输。 `SVC` 流在多个 `RTP` 会话中传输的传输模式。`RTP` 会话之间的依赖性必须通过本备忘录第7.2.3节的规定被通知。

`NAL` 单元解码顺序。 必须符合[H.264]G.7.4.1.2节中关于 `NAL` 单位顺序的限制条件。

`NALU-time`。 如果 `NALU` 在自己的 `RTP` 数据包中被传送就一定会包含的 `RTP` 时间戳的值。请注意，这个定义也存在于[RFC6184]中，形式完全相同。

操作点。 一个操作点由一组 `temporal_id`、`dependency_id` 和 `quality_id` 的值确定。  与操作点对应的比特流可以通过删除与较高的 `dependency_id` 值相关联的所有 `NAL` 单元，仅以有相同的 `dependency_id` 值但较高的 `quality_id` 或 `temporal_id` 值相关联的所有 `NAL` 单元来构造。一个操作点比特流符合[H.264]附件A 或 G中定义的至少一个 `profile`，并以一定的保真度提供原始视频信号的表示。

* 信息性说明：如果在特定的操作点上解码比特流不需要额外的 `NAL` 单元，则可以删除这些单元(具有较低的 `dependency_id` 或相同的 `dependency_id` 但较低的 `quality_id`)。然而，所产生的比特流可能不再符合[H.264]附件A或G中定义的任何 `profile`。

操作点表示。 同一接入单元内一个操作点的所有NAL单元的集合。

RTP数据包流。是指 `RTP` 数据包的序列，序列数不断增加(包绕除外)，有效载荷类型相同，`SSRC` (同步源)相同，在一个 `RTP` 会话中传输。 在本备忘录的中，一个RTP数据包流用来传输一层或多层数据。

单会话传输。 `SVC` 比特流通过一个 `RTP` 会话传输的传输模式。

`SVC` 基础层。 该层包括所有与 `dependency_id` 和 `quality_id` 值都等于0相关的 `NAL` 单元，包含前缀 `NAL` 单元（ `NAL` 单元类型14）。

`SVC` 增强层。`dependency_id` 或 `quality_id` 的值中至少有一个大于0的层。使用最大的 `dependency_id` 和 `quality_id` 值以及与 `SVC` 增强层相关联的任何 `temporal_id` 值构建的操作点不符合[H.264]附件A中规定的任何 `profile`。 

`SVC NAL`单元。 [H.264]附件G中规定的`14、15` 或 `20` 型 `NAL` 单元。

`SVC NAL` 单元头部。 在 `NAL` 单元类型 `14` 和 `20` 中增加了3个字节的 `SVC` 专用头扩展，产生了一个最终大小为4字节的头。

`SVC RTP` 会话。 基本 `RTP` 会话或增强 `RTP` 会话。

`T0 AVC` 基础层。 `AVC` 基础层的一个子集，它是由删除所有与 `temporal_id` 值大于0相关的`VCL NAL` 单元和仅与被删除的 `VCL NAL` 单元相关的非 `VCL NAL` 单元和 `SEI` 消息构成的。

`T0 SVC` 基础层。 `SVC` 基础层的一个子集，它是由删除所有与 `temporal_id` 值大于0相关的`VCL NAL` 单元以及前缀 `NAL` 单元、非 `VCL NAL` 单元和仅与被删除的 `VCL NAL` 单元相关的`SEI` 信息构成的。

传输顺序。 按 `RTP` 序列号升序排列的数据包顺序（按模数运算）。在一个聚合数据包中，`NAL` 单位的传输顺序与数据包中 `NAL` 单位的出现顺序相同。请注意，这个定义也存在于[RFC6184]中，其形式完全相同。

## 3.2.  Abbreviations

除[RFC6184]中定义的缩写外，本备忘录中还使用了以下缩写。

      CGS:        Coarse-Grain Scalability
      CS-DON:     Cross-Session Decoding Order Number
      MGS:        Medium-Grain Scalability
      MST:        Multi-Session Transmission
      PACSI:      Payload Content Scalability Information
      SST:        Single-Session Transmission
      SNR:        Signal-to-Noise Ratio
      SVC:        Scalable Video Coding

# 4.  RTP Payload Format

## 4.1.  RTP Header Usage

除了[RFC6184]第5.1节外，还适用以下规则。

* `M` 位的设置。如果一个 `RTP` 数据包的有效载荷是 `NI-MTAP`，且数据包中包含了与 `RTP` 时间戳相关联的接入单元按解码顺序的最后一个的 `NAL` 单元，，`M` 位必须等于1。

* 设置 `RTP` 时间戳。对于一个`RTP` 数据包，如果数据包的有效载荷是一个空 `NAL` 单元，`RTP` 的时间戳必须按照4.10节的规定进行设置。对于一个`RTP`数据包，如果数据包的有效载荷是一个 `PACSI NAL` 单元，`RTP` 时间戳必须等于传输顺序中下一个非 `PACSI NAL` 单元的 `NALU-time` 。  回顾 `MTAP` 中 `NAL` 单元的 `NALU-time` 在[RFC6184]中被定义为 `RTP` 时间戳的值，如果该 `NAL` 单元将在它自己的 `RTP` 包中传输。

* 设置SSRC。对于 `SST` 和 `MST`，`SSRC` 值必须根据[RFC3550]来设置。

## 4.2.  NAL Unit Extension and Header Usage

### 4.2.1.  NAL Unit Extension

本备忘录规定了一种 `NAL` 单元扩展机制，允许在[RFC6184]中剩下的三种未定义的NAL单元类型（即0、30和31）之外，引入新的NAL单元类型。扩展机制利用 `NAL` 单元类型值 `31`，具体如下。当 `NAL` 单位类型值等于 `31` 时，由第1.1.3节规定的 `F、NRI` 和 `Type` 字段组成的一字节 `NAL` 单位头扩展一个附加八位字节，它分别由一个名为 `Subtype` 的5位字段和三个名为 `J、K` 和 `L` 的1位字段组成。额外的八位组如下图所示。

         +---------------+
         |0|1|2|3|4|5|6|7|
         +-+-+-+-+-+-+-+-+
         | Subtype |J|K|L|
         +---------------+

`Subtype` 值决定了这个 `NALU`（扩展的）类型。字段 `J、K` 和 `L` 的解释取决于 `Subtype`。这些字段的语义会在下面给出。

当 `Subtype` 等于 `1` 时，NAL单元为4.10节规定的空 `NAL` 单元。  当 `Subtype` 等于 `2` 时，`NAL` 单元是第4.7.1节规定的 `NI-MTAP NAL` 单元。`Subtype (0, 3-31)` 的所有其他值都是为将来的扩展而保留的，当 `Subtype` 等于任何这些保留值时，接收机必须忽略整个 `NAL` 单元。

### 4.2.2.  NAL Unit Header Usage

在1.1.3节中介绍了根据H.264规范[H.264]的 `NAL` 单元头的结构和语义。本节根据本备忘录规定了 `NAL` 单元头字段 `F、NRI、I、PRID、DID、QID、TID、U` 和 `D` 的扩展语义。当 `Type` 字段等于 `31` 时，扩展 `NAL` 单元头中各字段的语义在Section4.2.1中规定。

[RFC6184]第5.3节中规定的 `F` 的语义也适用于本备忘录。也就是说，`F` 的值为 `0` 表示 `NALU` 类型八位组和有效载荷不应包含位错误或其他违规语法，而 `F` 的值为1表示 `NALU` 类型八位组和有效载荷可能包含位错误或其他违规语法。`MANEs` 应该设置 `F` 位表示 `NAL` 单元中的位错误。

对于 `NRI` ，对于符合[H.264]附件A定义的 `profile` 之一并使用[RFC6184]传输的比特流，[RFC6184]第5.3节规定的语义适用，即 `NRI` 也指示 `NAL` 单位的相对重要性。对于符合[H.264]附件G中定义的一个 `profile` 并使用本备忘录传输的比特流，除了[H.264]附件G中规定的语义外，`NRI` 还指示层中 `NAL` 单元的相对重要性。

对于 `I`，除了[H.264]附件G中规定的语义外，根据本备忘录，`MANEs` 可以利用该信息来保护 `I` 等于 `1` 的 `NAL` 单元，而不是 `I` 等于 `0` 的 `NAL` 单元，`MANEs` 还可以利用 `I` 等于 `1` 的 `NAL` 单元的信息来决定何时转发更多的 `RTP` 数据包流。例如，当检测到发生了空间层切换，使操作点变为更高的 `DID` 的值时，`MANEs` 可能只有在转发了 `I` 等于 `1` 的且 `DID` 较高值的 `NAL` 单元后，才开始转发 `DID` 较高值的 `NAL` 单元。

请注意，在本节的上下文中，"保护一个 `NAL` 单元 "是指任何可以提高交付 `NAL` 单元的数据包的成功率的 `RTP` 或网络传输机制，包括在可能的情况下，使用支持服务质量 (`QoS`) 的网络、前向纠错 (`FEC`)、重传和高级调度行为。

对于 `PRID` ，适用[H.264]附件G中规定的语义。注意，实施不平等错误保护的 `MANE` 可以利用这一信息来保护 `PRID` 值较小的 `NAL` 单元，例如，在 `FEC` 保护机制中只包括较重要的 `NAL` 单元。随着 `PRID` 值的增加，对解码过程的重要性也会降低。

对于 `DID、QID 或 TID`，除了[H.264]附件G中规定的语义外，根据本备忘录，`DID、QID 或 TID` 的值表示各自维度中的相对重要性。在其他两个组件是相同的情况下, 如果 `DID、QID 或 TID的` 值越低，表示重要性越。`MANEs` 可以使用此信息来保护较重要的 `NAL` 单位。

对于 `U` ，除了[H.264]附件G中规定的语义外，根据本备忘录，`MANEs` 使用该信息来保护 `U` 等于 `1` 的 `NAL` 单元，会比保护值为 `0` 的单元更好一些。

对于 `D`，除了[H.264]附件G中规定的语义外，根据本备忘录，`MANEs` 可以使用该信息来确定是否需要一个给定的 `NAL` 单元来成功解码 `SVC` 比特流的某个操作点，从而决定是否转发该 `NAL` 单元。

## 4.3.  Payload Structures

`NAL` 单元结构是 `H.264/AVC`、[RFC6184]、`SVC` 和本备忘录的核心。在 `H.264/AVC` 和 `SVC` 中，所有代表视频信号的编码位都封装在 `NAL` 单元中。在[RFC6184]中，每个 `RTP` 数据包的有效载荷结构为一个 `NAL` 单元，它包含了 `H.264/AVC` 中指定的一个 `NAL` 单元的一个或一部分，或集合了 `H.264/AVC` 中指定的一个或多个 `NAL` 单元。

[RFC6184]规定了三种基本的有效载荷结构(在[RFC6184]第5.2节中)：单 `NAL` 单元数据包、聚合数据包、分片单元，和六种新的 `NAL` 单元类型(24～29)。`RTP` 包有效载荷头的 `Type` 字段的值(即有效载荷的第一个字节)对于单个 `NAL` 单元包来说，可以等于 `1～23` 的任意值，对于聚合包来说，可以等于 `24～27` 的任意值，对于分片单位来说，可以等于 `28` 或 `29`。

除了最初为 `H.264/AVC` 定义的 `NAL` 单元类型外，还专门为 `SVC` 定义了三种新的 `NAL` 单元类型：可扩展扩展 `NAL` 单元中的编码片(类型 `20` )、前缀 `NAL` 单元(类型 `14` )和子集序列参数集 `NAL` 单元(类型 `15` )，如1.1节所述。

本备忘录进一步介绍了三种新类型的 `NAL` 单元，即第4.9节规定的 `PACSI NAL` 单元( `NAL` 单元类型 `30`)、第4.10节规定的空 `NAL` 单元(类型31，子类型1)和第4.7.1节规定的 `NI-MTAP NAL` 单元 (类型31，子类型2)。

在本备忘录中，保留了[RFC6184]中的 `RTP` 数据包有效载荷结构，并作了如下少量扩展。每个 `RTP` 数据包的有效载荷仍然是一个 `NAL` 单元的结构，它包含 `H.264/AVC` 和 `SVC` 中指定的一个 `NAL` 单元的一个或一部分，或包含一个 `PACSI NAL` 单元或一个空的 `NAL` 单元，或集合了 `H.264/AVC` 和 `SVC` 中指定的零个或多个 `NAL` 单元，零个或一个 `PACSI NAL` 单元，以及零或多个空的 `NAL` 单元。

在本备忘录中，三个基本有效载荷结构中的一个，即分段单元，与[RFC6184]中的结构保持不变，另外两个，即单 `NAL` 单元包和聚合包，扩展如下。单个 `NAL` 单元包的有效载荷头的 `Type` 字段的值可以等于 `1～23` (含)和 `30～31` (含)的任意值，聚合包的 `Type` 字段的值可以等于 `24～27` (含)和 `31` 的任意值。 当有效载荷头的 `Type` 字段等于 `31` 且有效载荷头的 `Subtype` 字段等于 `2` 时，该数据包为聚合数据包(包含一个 `NI-MTAP NAL` 单元)。 当有效载荷头的 `Type` 字段等于 `31` ，有效载荷头的 `Subtype` 字段等于 `1` 时，该数据包为单个 `NAL` 单元数据包（包含一个空 `NAL` 单元）。

请注意，在本备忘录中，有效载荷头的长度根据 `RTP` 包有效载荷第一个字节中的类型字段的值而变化。  如果该值等于14、20或30，则数据包有效载荷的前四个字节构成有效载荷头；否则，如果该值等于31，则有效载荷的前两个字节构成有效载荷头；否则，有效载荷头就是数据包有效载荷的第一个字节。

表1列出了 `SVC` 和本备忘录中介绍的 `NAL` 单元类型，以及本备忘录中对它们的描述。表2总结了所有 `NAL` 单元类型根据本备忘录直接用作 `RTP` 包有效载荷时的基本 `payload` 结构类型。表3总结了根据本备忘录允许聚合(即在聚合包中作为聚合单元使用)或分段(即在分段单元中携带)的 `NAL` 单元类型。

表1. `SVC` 和本备忘录中引入的 `NAL` 单位类型

    Type  Subtype  NAL Unit Name                Section Numbers
    -----------------------------------------------------------
    14     -       Prefix NAL unit                    1.1
    15     -       Subset sequence parameter set      1.1
    20     -       Coded slice in scalable extension  1.1
    30     -       PACSI NAL unit                     4.9
    31     0       reserved                           4.2.1
    31     1       Empty NAL unit                     4.10
    31     2       NI-MTAP                            4.7.1
    31     3-31    reserved                           4.2.1

表2.所有 `NAL` 单元类型直接用作 `RTP` 数据包有效载荷时的基本有效载荷结构类型。

    Type   Subtype    Basic Payload Structure
    ------------------------------------------
    0      -          reserved
    1-23   -          Single NAL Unit Packet
    24-27  -          Aggregation Packet
    28-29  -          Fragmentation Unit
    30     -          Single NAL Unit Packet
    31     0          reserved
    31     1          Single NAL Unit Packet
    31     2          Aggregation Packet
    31     3-31       reserved

表3. 允许合并或分割的 `NAL` 单位类型汇总(是=允许，否=不允许，-=不适用/未指定)

    Type  Subtype STAP-A STAP-B MTAP16 MTAP24 FU-A FU-B NI-MTAP
    -------------------------------------------------------------
    0     -          -      -      -      -     -     -     -
    1-23  -        yes    yes    yes    yes   yes   yes   yes
    24-29 -         no     no     no     no    no    no    no
    30    -        yes    yes    yes    yes    no    no   yes
    31    0          -      -      -      -     -     -     -
    31    1        yes     no     no     no    no    no   yes
    31    2         no     no     no     no    no    no    no
    31    3-31       -      -      -      -     -     -     -

## 4.4.  Transmission Modes

本备忘录可以在一个或多个 `RTP` 会话上传输 `SVC` 比特流，如果只有一个 `RTP` 会话用于传输 `SVC` 比特流，则传输模式称为单会话传输（`SST`）；否则（多个`RTP`会话用于传输`SVC`比特流），传输模式称为多会话传输(`MST`)。

`SST` 应该用于点对点的单播场景，而 `MST` 应该用于点对多点的多播场景，在这些场景中，不同的接收端对同一 `SVC` 比特流需要不同的操作点，以提高带宽利用效率。

如果可选性的 `mst-mode` 媒体类型参数（见7.1节）不存在，则必须使用 `SST` ；否则（`mst-mode`存在），必须使用`MST`。

## 4.5.  Packetization Modes

### 4.5.1.  Packetization Modes for Single-Session Transmission

当使用 `SST` 时，[RFC6184]第5.4节适用，并有以下扩展。

在[RFC6184]第5.4节中规定的分组模式，即单 `NAL` 单元模式、非交错模式和交错模式，也被称为会话分组模式。表4总结了 `SST` 允许的会话分组模式。

表4. `SST` 允许的会话打包模式摘要(为简化起见称为 "会话模式")(是=允许，否=不允许)。

    Session Mode               Allowed
    -------------------------------------
    Single NAL Unit Mode         yes
    Non-Interleaved Mode         yes
    Interleaved Mode             yes

对于 `0～29`（含）范围内的 `NAL` 单位类型，允许直接用作每个会话封装模式的数据包有效载荷的`NAL`单位类型与[RFC6184]第5.4节的规定相同。对于在本备忘录中新引入的其他 `NAL` 单元类型，允许直接用作每个会话封装模式的数据包有效载荷的 `NAL` 单元类型总结在表5中。

表5.  新的 `NAL` 单位类型，允许直接用作每个会话封装模式的数据包有效载荷(是=允许，否=不允许，-=不适用不指定)

    Type   Subtype    Single NAL    Non-Interleaved    Interleaved
                      Unit Mode           Mode             Mode
    -------------------------------------------------------------
    30     -            yes               no               no
    31     0              -                -                -
    31     1            yes              yes               no
    31     2             no              yes               no
    31     3-31           -                -                -

### 4.5.2.  Packetization Modes for Multi-Session Transmission

对于 `MST`，本备忘录规定了四种封装模式。

* 基于时间戳的非交错模式( `NI-T` );

* 基于交叉会话解码顺序号( `CS-DON` )的非交错模式( `NI-C` );

* 基于时间戳和 `CS-DON` 的非交错组合模式( `NI-TC` );

* 基于交错 `CS-DON(I-C)` 模式。

这四种模式有两方面的不同。首先，它们的区别在于 `NAL` 单元是需要在每个 `RTP` 会话内按解码顺序传输(即非交错)，还是允许它们以不同的顺序传输(即交错)。

其次，它们提供的机制不同，以便在所有涉及的 `RTP` 会中恢复 `NAL` 单元的正确解码顺序。

`NI-T、NI-C 和 NI-TC` 模式不允许交织，因此针对的是需要相对较低端到端延迟的系统，如对话系统。`I-C` 模式允许交织，因此针对的是不需要很低端到端延迟的系统。交错模式的优点与[RFC6184]中指定的交错模式相同。

`NI-T` 模式使用时间戳来恢复 `NAL` 单元的解码顺序，而 `NI-C` 和 `I-C` 模式都使用 `CS-DON` 机制(后面会解释)来恢复。  `NI-TC` 模式同时提供了时间戳和 `CS-DON` 方法；在这种情况下，接收机可以选择使用这两种方法来执行解码顺序恢复。使用的 `MST` 封装模式必须由可选项 ` mst-mode` 媒体类型参数的值来指定。所使用的 `MST` 打包模式规定了在相关的 `RTP` 会话中允许使用哪些会话打包模式，而这些模式又规定了哪些 `NAL` 单元类型被允许直接用作 `RTP` 数据包的有效载荷。

表6总结了 `NI-T、NI-C和NI-TC` 允许的会话打包模式。表7总结了 `I-C` 的允许会话打包模式。

表6. `NI-T、NI-C和NI-TC` 允许的会话打包模式概要(为简化起见称为 "会话模式")(是=允许，否=不允许)。

    Session Mode            Base Session    Enhancement Session
    -----------------------------------------------------------
    Single NAL Unit Mode         yes             no
    Non-Interleaved Mode         yes            yes
    Interleaved Mode              no             no

表7. `I-C` 允许的会话打包模式摘要(为简化起见，称为 "会话模式")(是=允许，否=不允许)

    Session Mode            Base Session    Enhancement Session
    -----------------------------------------------------------
    Single NAL Unit Mode          no             no
    Non-Interleaved Mode          no             no
    Interleaved Mode             yes            yes

对于 `0～29` （含）范围内的 `NAL` 单位类型，允许直接用作每个会话封装模式的数据包有效载荷的 `NAL` 单位类型与[RFC6184]第5.4节中的规定相同。对于本备忘录中新引入的其他 `NAL` 单元类型， `NI-T、NI-C、NI-TC和I-C` 的每一种允许的会话封装模式中，允许直接用作数据包有效载荷的NAL单元类型分别汇总在表8、表9、表10和表11中。

表8. 当使用 `NI-T` 时，允许新的 `NAL` 单位类型直接用作每个允许的会话封装模式的数据包有效载荷(是=允许，否=不允许，-=不适用/未指定)

    Type   Subtype    Single NAL    Non-Interleaved
                      Unit Mode           Mode
    ---------------------------------------------------
    30     -            yes               no
    31     0              -                -
    31     1            yes              yes
    31     2             no              yes
    31     3-31           -                -

表9.当使用 `NI-C` 时，允许新的 `NAL` 单位类型直接用作每个允许的会话封装模式的数据包有效载荷(是=允许，否=不允许，-=不适用/未指定)

    Type   Subtype    Single NAL    Non-Interleaved
                      Unit Mode           Mode
    ---------------------------------------------------
    30     -            yes              yes
    31     0              -                -
    31     1             no               no
    31     2             no              yes
    31     3-31           -                -

表10.当使用 `NI-TC` 时，允许新的 `NAL` 单位类型直接用作每个允许的会话封装模式的数据包有效载荷(是=允许，否=不允许，-=不适用/未指定)

    Type   Subtype    Single NAL    Non-Interleaved
                      Unit Mode           Mode
    ---------------------------------------------------
    30     -            yes              yes
    31     0              -                -
    31     1             yes             yes
    31     2             no              yes
    31     3-31           -                -

表11.当使用 `I-C` 时，允许新的 `NAL` 单位类型直接用作每个允许的会话封装模式的数据包有效载荷(是=允许，否=不允许，-=不适用/未指定)

    Type   Subtype    Interleaved Mode
    ------------------------------------
    30     -               no
    31     0                -
    31     1               no
    31     2               no
    31     3-31             -

当使用 `MST` ，且使用的 `MST` 打包模式为 `NI-C` 时，必须不使用空 `NAL` 单元(类型31，子类型1)，即不允许 `RTP` 包包含一个或多个空 `NAL` 单元。

当使用 `MST`，且使用的 `MST` 打包模式为 `I-C` 时，必须不能使用空 `NAL` 单元(类型31,子类型1)和 `NI-MTAP NAL` 单元(类型31,子类型2)，即不允许 `RTP` 数据包包含一个或多个空 `NAL` 单元或 `NI-MTAP NAL` 单元。

## 4.6.  Single NAL Unit Packets

[RFC6184]第5.6节适用于以下扩展。

单个 `NAL` 单元数据包的有效载荷可以是一个 `PACSI NAL` 单元(Type 30)或一个空 `NAL` 单元(Type 31 和 Subtype 1)，此外还有一个 `NAL` 单元，或者是 `NAL` 单元类型等于1至23的任何值的单元。

如果有效载荷的第一个字节的类型字段不等于 `31`，则有效载荷头是有效载荷的第一个字节，否则，有效载荷头是有效载荷的前两个字节。

## 4.7.  Aggregation Packets

除[RFC6184]第5.7节外，以下内容适用于本备忘录。

### 4.7.1.  Non-Interleaved Multi-Time Aggregation Packets (NI-MTAPs)

本备忘录中引入的一种新的 `NAL` 单元类型是非交错多时聚合包（`NI-MTAP`）。  一个 `NI-MTAP` 由一个或多个非交错多时聚合单元组成。

`NI-MTAPs` 中包含的 `NAL` 单元必须按解码顺序汇总。

`NI-MTAP` 的一个非交错多时聚合单元由以下 `NAL` 单元的16位无符号 `size` 信息(按网络字节顺序)和 `NAL` 单元的 `16` 位(按网络字节顺序)时间戳偏移( `TS` 偏移)组成。其结构如图1所示。数据包中一个聚合单元的起始或结束位置可能在 `32` 位字边界上，也可能不在。`NI-MTAP` 中的 `NAL` 单元是按 `NAL` 单元解码顺序排列的。

`NI-MTAP` 的 `Type` 字段必须设置为 "31"。

如果聚集的 `NAL` 单元的所有 `F` 位都是0，则 `F` 位必须设置为0；否则，它必须设置为1。

`NRI` 的值必须是 `NI-MTAP` 数据包中携带的所有 `NAL` 单元的最大 `NRI` 值。

字段 `Subtype` 必须等于2。

如果字段 `J` 等于 `1` ，那么对于每个非交错多时聚合单元来说，可选的 `DON` 字段必须存在，对于 `SST` 来说，`J` 字段必须等于 `0`，对于 `MST` 来说，在 `NI-T` 模式下，`J` 字段 `MUST` 等于 `0` ，而在 `NI-C` 或 `NI-TC` 模式下，`J` 字段必须等于1。当使用 `NI-C` 或 `NI-TC` 模式时，且存在 `DON` 字段时，该字段必须代表第6.2.2节中定义的特定 `NAL` 单元的 `CS-DON`值。

字段 `K` 和 `L` 必须都等于 `0`。

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    :        NAL unit size          |        TS offset              |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |        DON (optional)         |                               |
    |-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+    NAL unit                   |
    |                                                               |
    |                               +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                               :
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

图1.`NI-MTAP` 非交错多时聚合单元

以 `TS` 作为 `NAL` 单元的数据包的 `RTP` 时间戳.回顾一下在 `MTAP` 中， `NAL` 单元的 `NALU-time` 在[RFC6184]中被定义为如果该 `NAL` 单元在自己的 `RTP` 数据包中传输时 `RTP` 时间戳的值。`timestamp offset` 字段必须设置为等于下面公式的值:

      if NALU-time >= TS, TS offset = NALU-time - TS
      else, TS offset = NALU-time + (2^32 - TS)

对于 `NI-MTAP` 中 "最早 "的多时间聚合单元，时间戳偏移必须为零。因此，`NI-MTAP` 本身的 `RTP` 时间戳与最早的 `NALU-time` 相同。

* 信息性说明："最早" 的多时间聚合单元是指在 `NI-MTAP` 的所有聚合单元中具有最小的扩展 `RTP` 时间戳的单元，如果聚合单元被封装在单个 `NAL` 单元数据包中。扩展时间戳是指具有32位以上的时间戳，并且能够计算时间戳字段的环绕，从而使人们能够确定时间戳环绕时的最小值。  这样一个 "最早 "聚合单元可以是也可以不是聚合单元被封装在 `NI-MTAP` 中的顺序中的第一个单元。"最早 "的 `NAL` 单元也不需要与 `NAL` 单元解码顺序中的第一个 `NAL` 单元相同。

图2是一个包含 `NI-MTAP` 的 `RTP` 数据包的例子，该数据包包含两个非交错的多时间聚合单元，图中标为1和2。

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                          RTP Header                           |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |F|NRI|  Type   | Subtype |J|K|L|                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                               |
    |                                                               |
    |        Non-interleaved multi-time aggregation unit #1         |
    :                                                               :
    |                                 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                 |  Non-interleaved multi-time |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                             |
    |                      aggregation unit #2                      |
    :                                                               :
    |                               +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                               :...OPTIONAL RTP padding        |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

图2.一个包含 `NI-MTAP` 的 `RTP` 数据包，其中包含两个交错的多时间聚合单元。

## 4.8.  Fragmentation Units (FUs)

[RFC6184]的5.8节适用。

* 信息性说明：如果一个具有4字节 `SVC NALU` 头的 `NAL` 单元被分段，`3` 字节 `SVC` 专用头扩展被视为 `NAL` 单元有效载荷的一部分。也就是说，三字节的 `SVC` 专用头扩展只在分段 `NAL` 单元的第一个分段中可用。

## 4.9.  Payload Content Scalability Information (PACSI) NAL Unit

本备忘录中规定的另一种新的 `NAL` 单元类型是载荷内容可扩展性信息`（PACSI）NAL` 单元。`PACSI NAL` 单元的类型字段必须等于 `30`（在[H.264]和[RFC6184]中未指定的NAL单元类型值）。一个 `PACSI NAL` 单元可以在单个 `NAL` l单元数据包或聚合数据包中承载，并且必须不被分段。

`PACSI NAL` 单元可用于以下目的：

* 使 `MANEs` 能够决定是否转发、处理或丢弃聚合数据包，方法是在 `PACSI NAL` 单元中检查聚合 `NAL` 单元的可扩展性信息和其他特性，而不是查看聚合 `NAL` 单元本身，后者是由视频编解码规范定义的； 

* 借助 `PACSI NAL` 单元中包含的 `CS-DON` 信息，使用 `NI-C或NI-TC` 模式，在 `MST` 中实现正确的解码顺序恢复；

* 利用 `PACSI NAL` 单元中包含的以下数据或信息：重复的补充增强信息(SEI)消息，有关层表示的开始和结束的信息，以及最低时间子集的层表示的索引，提高丢包的恢复能力。

在 `NI-T` 模式下可以忽略 `PACSI NAL` 单元，而不影响解码顺序恢复过程。

当聚合数据包中存在 `PACSI NAL` 单元时，以下内容适用。

* `PACSI NAL` 单元必须是聚合数据包中的第一个聚合 `NAL` 单元。

* 在聚合数据包中必须有至少一个额外的聚合 `NAL` 单元。

* 聚合数据包的 `RTP` 头字段和有效载荷头字段的设置就像聚合数据包中没有包含 `PACSI NAL` 单元一样。

* 如果聚合数据包是 `MTAP16、MTAP24或NI-MTAP`，且 `J` 字段等于 `1`，则必须设置 `PACSI NAL` 单元的解码顺序号(DON)，以表明`PACSI NAL`单元与聚合数据包中剩余的 `NAL`单元中解码顺序中的第一个NAL单元具有相同的 `DON`。

当一个`PACSI NAL`单元被包含在单个`NAL`单元数据包中时，它将与下一个按传输顺序排列的非`PACSI NAL`单元相关联，数据包的`RTP`头字段的设置，与下一个被包含在单个`NAL`单元数据包中的按传输顺序排列的非`PACSI NAL`单元一样。

`PACSI NAL`单元结构如下。  前四个八位字节与1.1.3节中讨论的四字节 `SVC NAL` 单元头完全相同。  它们后面是一个包含多个标志的八位字节，然后是五个可选的八位字节，最后是零个或多个`SEI NAL`单元。  每个`SEI NAL`单元前都有一个16位无符号大小字段(按网络字节顺序)，它以字节为单位表示下一个`NAL` 单元的大小(不包括这两个八位字节，但包括`SEI NAL`单元的 `NAL` 单元头部八位字节)。图3说明了`PACSI NAL`单元结构和一个包含两个`SEI NAL`单元的`PACSI NAL`单元的例子。

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |F|NRI|  Type   |R|I|   PRID    |N| DID |  QID  | TID |U|D|O| RR|
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |X|Y|T|A|P|C|S|E| TL0PICIDX (o) |        IDRPICID (o)           |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |          DONC (o)             |        NAL unit size 1        |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    |                 SEI NAL unit 1                                |
    |                                                               |
    |               +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |               |        NAL unit size 2        |               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+               |
    |                                                               |
    |            SEI NAL unit 2                                     |
    |                               +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

图3. `PACSI NAL`单元结构。以"(o)" 为后缀的字段为可选字段。

只有当位`X`等于 `1` 时，才会指定位`A、P和C`。只有当位`Y`等于 `1` 时，才会指定位`S和E`，并存在`TL0PICIDX` 和 `IDRPICID` 字段。 只有当位`T`等于 `1` 时，才会出现字段 `DONC`。如果 `PACSI NAL` 单元包含在 `STAP-B、MTAP16、MTAP24或NI-MTAP` 中，且 `J` 字段等于 `1`，则字段 `T` 必须等于 `0`。

`PACSI NAL` 单位的字段值必须设置如下：

略

## 4.10.  Empty NAL unit

空 `NAL` 单元可以包含在单个 `NAL` 单元数据包、`STAP-A或NI-MTAP` 数据包中。空的 `NAL` 单元必须有一个 `RTP` 时间戳（在单个NAL单元包中传输时）或 `NALU-time`（在聚合包中传输时），该 `RTP` 时间戳和 `NALU-time` 与至少存在一个类型为 `1、5或20` 的 `NAL` 单元的接入单元相关联。  当使用 `MST` 时，类型 `1、5或20` 的 `NAL` 单元可能在不同的 `RTP` 会话中。空 `NAL` 单元可用于 `NI-T` 模式的解码顺序恢复过程，如第5.2.1节所述。

数据包结构如下图所示。

    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |F|NRI|  Type   | Subtype |J|K|L|
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

图4. 空的NAL单元结构

这些字段必须设置如下：

## 4.11.  Decoding Order Number (DON)

`DON` 概念是在[RFC6184]中引入的，当在单个会话中使用交织时，`DON` 概念用于恢复解码顺序.当使用 `SST` 时，[RFC6184]的5.5节适用。

当使用 `MST` 时，无论是否使用交织，都需要恢复各个 `RTP` 会话的解码顺序。除了后面描述的时间戳机制外，`CS-DON` 机制是 `DON` 设施的扩展，可以用于此目的，并在下一节中定义。

### 4.11.1.  Cross-Session DON (CS-DON) for Multi-Session Transmission

跨会话解码顺序号(`CS-DON`)是一个表示 `NAL` 单元在所有 `RTP` 会话中的解码顺序的数字。它类似于[RFC6184]中的 `DON` 概念，但与[RFC6184]中的 `DON` 只用于交错打包不同，在本备忘录中，它不仅用于交错 `MST` 模式(`I-C`)，还用于两种非交错`MST`模式(`NI-C和NI-TC`)。

当使用`NI-C`或`NI-TC MST`模式时，每个会话的分组必须符合第5.2.2节的规定。在`PACSI NAL`单元中，`CS-DON`值被明确地编码在`DONC`字段中。对于非`PACSI NAL`单元，`CS-DON`值的推导方式如下。 这里以 `SN` 表示一个数据包的 `RTP` 序列号。

* 对于使用单 `NAL` 单元会话分组模式的会话中所携带的每个非 `PACSI NAL` 单元，`NAL`单元的`CS-DON` 值等于 `(DONC_prev_PACSI + SN_diff - 1) % 65536`，其中 "%" 是模数运算，`DONC_prev_PACSI` 是与当前 `NAL` 单元具有相同 `NALU` 时间的前一个 `PACSI NAL` 单元的 `DONC` 值，`SN_diff` 的计算方法如下。

      if SN1 > SN2, SN_diff = SN1 - SN2
      else SN_diff = SN2 + 65536 - SN1

  当 `SN1` 和 `SN2` 分别是当前 `NAL` 单元和上一个 `PACSI NAL` 单元的`SN`，具有相同的 `NALU` 时间。

* 对于使用非交错会话打包模式的会话中携带的非 `PACSI NAL` 单元，每个非 `PACSI NAL` 单元的`CS-DON` 值如下。

  * 对于单个`NAL`单元包中的非`PACSI NAL`单元，以下情况适用。

    * 如果前一个 `PACSI NAL` 单元包含在一个 `NAL` 单元包中，则 `NAL` 单元的 `CS-DON` 值如上计算；

    * 否则(前一个 `PACSI NAL` 单元包含在一个 `STAP-A` 包中)，`NAL` 单元的 `CS-DON` 值如上计算，`DONC_prev_PACSI` 被前一个非 `PACSI NAL` 单元的 `CS-DON值` 代替，以不编码顺序(即`STAP-A` 包中最后一个 `NAL` 单元的 `CS-DON` 值)代替。 即 `STAP-A` 包最后一个 `NAL1` 单元的 `CS-DON` 值）。)

  * 对于 `STAP-A` 数据包中的非 `PACSI NAL` 单元，以下内容适用：

    * 如果非 `PACSI NAL` 单元是 `STAP-A` 数据包中的第一个非 `PACSI NAL`单元，则该 `NAL` 单元的 `CS-DON` 值等于 `STAP-A` 数据包中 `PACSI NAL` 单元的 `DONC` ；否则，该 `NAL` 单元的 `CS-DON` 值等于: `(前一个非PACSI NAL单元的CS-DON值按解码顺序+1)% 65536`，其中"%"为模数运算。

  * 对于若干 `FU-A` 包中的非 `PACSI NAL` 单元，`NAL` 单元的 `CS-DON` 值的计算方法与使用单个 `NAL` 单元会话包化模式时相同，`SN1` 为第一个 `FU-A` 包的 `SN` 值。

  * 对于 `NI-MTAP` 数据包中的非 `PACSI NAL` 单元，`CS-DON` 值等于非交错多时聚合单元 `DON` 字段的值。

当使用 `I-C MST` 打包模式时，根据[RFC6184]得出的每个 `RTP` 会话中所有 `NAL` 单元的 `DON` 值必须表示 `CS-DON` 值。

# 5.  Packetization Rules

[RFC6184]第6节适用于本备忘录，但增加以下内容：

## 5.1.  Packetization Rules for Single-Session Transmission

所有的接收机都必须支持单 `NAL` 单位打包模式，以便向后兼容只支持[RFC6184]单 `NAL` 单位模式的终端，但是，只要有可能，就应该避免使用单NAL单元封装模式( `packetization-mode` 等于 `0` )，因为将小尺寸的 `NAL` 单元封装在自己的数据包中(例如，包含参数集、前缀 `NAL` 单元或 `SEI` 消息的小 `NAL` 单元)效率较低，因为数据包头开销较大。

所有接收机必须支持非交错模式。

* 信息性说明：[RFC6184]的非交错模式确实允许应用程序在单个 `RTP` 包中封装单个 `NAL` 单元。  历史上，[RFC6184]中包含单 `NAL` 单元模式只是为了与ITU-T Rec.H.241附件A[H.241]兼容。支持非交错模式的附加机制(即 `STAP-A和FU-A` )的实现复杂性增加不大，而好处却很大。因此，需要支持 `STAP-A和FU-A`。  此外，本备忘录中定义的三种 `NAL` 单元类型中的两种，即空 `NAL` 单元和 `NI-MTAP` 需要支持，具体见4.5.1节。

小尺寸的 `NAL` 单元应该与一个或多个其他 `NAL` 单元一起封装在聚合包中。例如，非 `VCL NAL` 单元，如访问单元定界符、参数集或 `SEI NAL` 单元，通常都是小单元。

前缀`NAL` 单元和与其相关联的 `NAL` 单元，以及按照解码顺序跟在前缀 `NAL` 单元后面的 `NAL` 单元，每当聚合包用于相关联的 `NAL` 单元时，都应该包含在同一个聚合包中，除非这样做会违反会话 `MTU` 约束或如果分段单元被用于相关联的 `NAL` 单元。

* 信息性说明：虽然前缀 `NAL` 单元被 `H.264/AVC` 解码器忽略，但它在 `SVC` 解码过程中是必要的。

考虑到前缀 `NAL` 单元的尺寸较小，最好将其与相关联的 `NAL` 单元在同一个RTP包中传输。

当在RTP会话中只传输 `SVC` 基础层的 `H.264/AVC` 兼容子集时，必须根据[RFC6184]对该子集进行封装。这样，[RFC6184]接收器就能接收到 `H.264/AVC` 兼容的比特流子集。

当一组包括一个或多个 `SVC` 增强层的层在 `RTP` 会话中传输时，该组应该在一个`RTP`流中进行，并根据本备忘录进行封装。

## 5.2.  Packetization Rules for Multi-Session Transmission

当使用 `MST` 时，第5.1节中规定的分组规则仍然适用。此外，必须遵循以下分组规则，以确保使用第6.2节规定的解包过程，正确恢复每个 `MST` 分组模式中 `NAL` 单元的解码顺序。

`NI-T和NI-TC` 模式都使用时间戳来恢复解码顺序。为了能够做到这一点，`RTP` 包流必须包含给定`RTP` 会话的所有采样实例的数据，这些数据在所有依赖于给定`RTP` 会话的增强 `RTP` 会话中。  `NI-C和I-C` 模式没有这种限制，使用 `CS-DON` 值作为明确指示解码顺序的手段，可以直接用`PACSI NAL` 单元编码，也可以使用包化规则从它们中推断。值得注意的是，`NI-TC` 模式提供了两种选择，由接收方选择使用哪一种。

### 5.2.1.  NI-T/NI-TC Packetization Rules

当使用 `NI-T` 模式且存在 `PACSI NAL` 单元时，`T` 位必须等于 `0`，即 `DONC` 字段必须不存在。

当使用 `NI-T` 模式时，可选的参数 `sprop-mst-remux-buf-size、sprop-remux-buf-req、remux-buf-cap、sprop-remux-init-buf-time、sprop-mst-max-don-diff` 必须不存在。

当使用 `NI-T或NI-TC MST` 模式时，适用以下情况。

如果在 `RTP` 会话 `A` 中存在一个或多个采样时间实例的接入单元的 `NAL` 单位，那么在任何依赖于 `RTP` 会话 `A` 的增强型 `RTP` 会话中必须存在一个或多个相同接入单元的 `NAL` 单位。

* 信息性说明：`RTP和NTP` 格式时间戳之间的映射是在 `RTCP SR` 数据包中传递的。此外，[RFC6051]中讨论的加快媒体时间戳同步的机制可用于加快 `RTP-to-wall-clock` 映射的获取。

* 信息性说明：上述规则可能需要插入 `NAL` 单元，通常是在使用时间可扩展性时，即一个增强 `RTP` 会话不包含具有特定 `NTP` 时间戳(媒体时间戳)的访问单元的任何 `NAL` 单元，但该访问单元存在于一个较低的增强 `RTP` 会话或基本 `RTP` 会话中。有两种方法可以插入额外的 `NAL` 单元以满足这一规则。

  - 增加额外的 `NAL` 单元的一种选择是使用空的 `NAL` 单元(在第4.10节中定义)，它可以被第6.2.1节中描述的过程用于访问单元重新排序过程。

  - 编码器本身也可以增加额外的 `NAL` 单元，例如，通过传输编码数据，简单地指示解码器重复上一张图片。然而，这种选择可能难以用于预先编码的内容。

如果必须插入一个数据包才能满足上述规则，例如，在一个 `MANEs` 从一个 `RTP` 流中产生多个 `RTP` 流的情况下，插入的数据包必须有一个 `RTP` 时间戳，该时间戳必须与任何低级增强 `RTP` 会话或基本 `RTP` 会话中存在的接入单元的任何数据包的 `RTP` 时间戳映射到相同的挂钟时间（NTP格式）。  如果 `NAL` 单元或数据包可以在 `RTP` 流生成时插入，这很容易实现，因为插入的数据包和相应接入单元的数据包的介质时间戳（NTP 时间戳）必须是相同的。如果在RTP流生成时不知道介质时间，或者RTP流不是在同一个实例中生成的，这也可以在传输过程的后期应用。在这种情况下，插入数据包的 `NTP` 时间戳可以按以下方式计算。

假设在基础 `RTP` 会话 `A` 中出现了一个带有 `RTP` 时间戳 `TS_A2` 的接入单元的数据包 `A2` ，而在增强 `RTP` 会话 `B` 中没有该接入单元的数据包，如图5所示，因此，必须按照上述规则在会话 `B` 中插入一个数据包 `B2` 。因此，必须按照上面的规则在会话 `B` 中插入一个数据包 `B2`，会话 `A` 中最近的 `RTCP` 发送者报告带有 `NTP` 时间戳 `NTP_A` 和 `RTP` 时间戳 `TS_A` 。会话 `B` 中 `NTP` 时间戳比 `NTP_A` 低的发件人报告为 `NTP_B`，并携带 `RTP` 时间戳 `TS_B`。

     RTP  session B:..B0........B1........(B2)......................
     RTCP session B:.....SR(NTP_B,TS_B).............................
     RTP  session A:..A0........A1........A2........................
     RTCP session A:..................SR(NTP_A,TS_A)................
     -----------------|--x------|-----x---|------------------------>
                                                              NTP time
     --------------------+<---------->+<->+------------------------>

                               t1       t2              RTP TS(B) time
图5.增强层 `RTP` 会话中数据包插入的 `RTP` 时间戳计算示例

上图中 `NTP` 时间线中的竖条("|")表示至少有一个会话中存在接入单元数据。 "x" 标记表示发送者报告的时间。`t1` 是两个会话之间发送者报告的时间差，用 `RTP` 时间戳时钟刻度表示，`t2` 是会话 `A` 发送者报告到 `A2` 数据包的时间差，同样用 `RTP` 时间戳时钟刻度表示。将这些差值之和加到来自会话 `B` 的会话报告的 `RTP` 时间戳上，以得出插入的数据包 `B2` 的正确 `RTP` 时间戳。 换句话说:

    TS_B2 = TS_B + t1 + t2
    
让 `toRTP()` 是计算给定 `NTP` 时间戳差的 `RTP` 时间差的函数(以所用时钟的时钟刻度为单位)，让 `effRTPdiff()` 是计算两个时间戳之间的有效差的函数：

    effRTPdiff( ts1, ts2 ):

        if( ts1 <= ts2 ) then
            effRTPdiff := ts1-ts2
        else
            effRTPDiff := (4294967296 + ts2) - ts1

可以得到：

    t1 = toRTP(NTP_A - NTP_B)
    
    t2 = effRTPdiff(TS_A2，TS_A)

因此，为了生成插入包 `B2` 的 `RTP` 时间戳 `TS_B2`，可以计算包 `B2` 的 `RTP` 时间戳 `TS_B2`，如下。

    TS_B2 =  TS_B + toRTP(NTP_A - NTP_B) +  effRTPdiff(TS_A2, TS_A)

### 5.2.2.  NI-C/NI-TC Packetization Rules

当使用 `NI-C` 或 `NI-TC MST` 模式时，以下内容适用于每个 `RTP` 会话。

* 对于每一个包含非 `PACSI NAL` 单元的单 `NAL` 单元数据包，前一个数据包如果存在，必须具有与单 `NAL` 单元数据包相同的 `RTP` 时间戳，并且以下内容适用。

  * 如果非 `PACSI NAL` 单元的 `NALU-time` 不等于前一个非 `PACSI NAL` 单元在解码顺序中的 `NALU-time` ，前一个数据包必须包含一个包含 `DONC` 字段的 `PACSI NAL` 单元。

* 在`STAP-A`数据包中，`STAP-A`数据包中的第一个`NAL`单元必须是包含`DONC`字段的`PACSI NAL`单元。

* 对于`FU-A`数据包，前一个数据包必须与`FU-A`数据包具有相同的RTP时间戳，并且以下情况适用：

  * 如果`FU-A`包是分段 `NAL` 单元的开始，则适用以下规定。

    * 如果分段 `NAL` 单元的 `NALU-time` 不等于前一个非 `PACSI NAL` 单元在解码顺序中的`NALU-time`，则前一个数据包必须包含一个包含 `DONC` 字段的 `PACSI NAL` 单元；

    * 否则，（分段 `NAL` 单元的 `NALU-time` 等于前一个非 `PACSI NAL` 单元在解码顺序中的`NALU-time`），前一个数据包可能会包含一个包含 `DONC` 字段的 `PACSI NALU`。

  * 否则，如果 `FU-A` 包是分段 `NALU` 的末端，则适用于以下情况。

    * 如果下一个解码顺序中的非 `PACSI NAL` 单元的 `NALU-time` 等于分段 `NAL` 单元的`NALU-time`，并且是以若干 `FU-A` 包或单个 `NAL` 单元包的形式携带，那么下一个包必须是一个包含 `DONC` 字段的 `PACSI NAL` 单元的单个 `NAL` 单元包。

    * 否则（`FU-A` 包既不是碎片化的 `NAL` 单元的开始，也不是结束），前一个包必须是 `FU-A` 包。

* 对于每一个包含 `PACSI NAL` 单元的单个 `NAL` 单元数据包，如果存在，`PACSI NAL` 单元必须包含 `DONC` 字段。

* 当可选的媒体类型参数 `sprop-mst-csdon-always-present` 等于 `1` 时，使用的会话分组模式必须是非交错模式，并且只能使用 `STAP-A和NI-MTAP` 数据包。

### 5.2.3.  I-C Packetization Rules

当使用 `I-C MST` 封装模式时，适用于以下情况。

当存在 `PACSI NAL` 单元时，`T` 位必须等于0，即 `DONC` 字段不存在，`Y` 位必须等于0，即 `TL0PICIDX` 和 `IDRPICID` 不存在。

### 5.2.4.  Packetization Rules for Non-VCL NAL Units

不直接对视频片进行编码的 `NAL` 单元在H.264中被称为非 `VCL NAL` 单元。仅由增强型 `RTP` 会话使用或仅与增强型 `RTP` 会话相关的非 `VCL` 单元应在与其相关的最低会话中发送。

然而，一些发送者，如发送预编码数据的发送者，可能无法轻易确定哪些非 `VCL` 单元与哪个会话相关。因此，非 `VCL NAL` 单元可以在使用这些非 `VCL NAL` 单元的会话所依赖的会话中发送(例如，基准 `RTP` 会话)。

如果一个非 `VCL` 单元与一个以上的 `RTP` 会话相关，而这些会话都不依赖于其他会话，那么 `NAL` 单元可以在所有这些会话都依赖于的另一个会话中发送。

### 5.2.5.  Packetization Rules for Prefix NAL Units

本备忘录第5.1节适用，但增加以下内容。如果基础层使用[RFC6184]在基础 `RTP` 会话中发送，前缀 `NAL` 单元可以在最低增强 `RTP` 会话中发送，而不是在基础 `RTP` 会话中发送。

# 6.  De-Packetization Process

## 6.1.  De-Packetization Process for Single-Session Transmission

对于单会话传输，如果使用单个RTP会话，则适用[RFC6184]第7节规定的解包过程。

## 6.2.  De-Packetization Process for Multi-Session Transmission

对于多会话传输，当一个以上的 `RTP` 会话用于接收来自同一 `SVC` 比特流的数据时，解包过程规定如下。

对于单个`RTP`会话，解包过程背后的一般概念是将 `NAL` 单元从传输顺序到 `NAL` 单元解码顺序重新排序。

要接收的会话必须由第7.2.3节中规定的机制来识别。一个增强型 `RTP` 会话通常包含一个`RTP`流， 且依赖于至少一个其他 `RTP` 会话，如第7.2.3节中定义的机制所示。 一个增强型 `RTP` 会话的更低级别`RTP` 会话是指增强型 `RTP` 会话所依赖的 `RTP` 会话。对于接收者来说，最低的RTP会话是基础RTP会话，它不依赖于接收者收到的任何其他 `RTP` 会话。对于接收者来说，最高的 `RTP` 会话是指所有 `RTP` 会话都不依赖的 `RTP` 会话。

对于每个 `RTP` 会话，应用RFC 3550中规定的 `RTP` 接收过程。然后将接收到的数据包传入本备忘录中定义的有效载荷解包过程。

所有相关的`RTP`会话中携带的 `NAL` 单元的解码顺序，然后通过应用下面的一个子节的规则来恢复，这取决于使用的是哪种 `MST` 封装模式。

### 6.2.1.  Decoding Order Recovery for the NI-T and NI-TC Modes

当使用 `NI-T` 封装模式时，必须应用以下过程。当使用 `NI-TC` 封装模式时，可以应用以下过程。

该过程基于 `RTP` 会话依赖性信令、`RTP` 序列号和时间戳。

`RTP` 会话中 `RTP` 包流内 `NAL` 单元的解码顺序由包含 `NAL` 单元的 `RTP包` 的序列号 `SN` 的顺序和包内 `NAL` 单元的出现顺序给出。

根据媒体时间戳 `TS`，即从 `RTP` 数据包的 `RTP` 时间戳得出的`NTP` 时间戳的计时信息，与 `RTP` 会话中接收到的同一个`RTP`数据包中包含的所有`NAL`单元相关联。

对于 `NI-MTAP` 数据包，通过使用 `NI-MTAP` 数据包中的 "TS偏移 "值（如第4.10节中的定义），为每个包含的 `NAL` 单元推导出 `NALU-time`，并代替 `RTP` 数据包时间戳来推导出媒体时间戳，例如，使用通过 `RTCP` 发送者报告提供的 `NTP` 挂钟。包含在分段数据包的 `NAL` 单元被处理为不分段，整个 `NAL` 单元有自己的媒体时间戳。所有与媒体时间戳 `TS` 的相同值相关联的 `NAL` 单元都是同一个访问单元 `AU(TS)` 的一部分。在重新排序过程中，任何空 `NAL` 单元都应该被有效地保留为访问单元指示器。空 `NAL` 单元和 `PACSI NAL` 单元应该在将访问单元数据传递给解码器之前被移除。

* 信息性说明：这些空 `NAL` 单元用于将其他 `RTP` 会话中存在的 `NAL` 单元与不包含特定时间实例的访问单元的任何数据的 `RTP` 会话联系起来。它们在会话中充当访问单元指示器，否则它们将不包含特定访问单元的数据。这些NAL单元的存在由第5.2.1节中的打包规则保证。

假设接收者已经建立了一个操作点 `(DID、QID 和 TID` 值)，并为这个操作点确定了最高增强的 `RTP` 会话。  在多个 `RTP` 会话中，来自多个 `RTP` 流的 `NAL` 单元的解码顺序必须通过执行以下步骤中的过程来恢复成一个单一的 `NAL` 单元序列，分组为访问单元，一般过程在[RFC6051]的4.2节中描述。为了方便起见，重复了[RFC6051]的指令，并将其应用于 `NAL` 单元，而不是完整的 `RTP` 数据包。此外，[RFC6051]第4.2.节中的程序的 `SVC` 特定扩展在下面的列表中介绍。

* 该过程应从最高的 `RTP` 会话中接收到的 `NAL` 单元开始，并在该会话的（去抖动）缓冲区中得到第一个媒体时间戳 `TS`（ `NTP` 格式）。假定去抖动缓冲区中的数据包已经按 `RTP` 序列号顺序存储。

* 从最高的 `RTP` 会话开始，从接收到的所有 `RTP` 会话的（去抖动）缓冲区中，收集与媒体时间戳 `TS` 相同值相关的所有 `NAL` 单元。收集的 `NAL` 单元将是与接入单元 `AU(TS)` 相关的单元。

* 按照第7.2.3节规定的依存度指示所得出的会话依存度顺序，从最低的 `RTP` 会话开始，放置收集到的 `NAL` 单元。

* 通过满足 `SVC` 接入单元的 `NAL` 单元顺序规则，将会话排序的 `NAL` 单元与特定接入单元放在一起解码，如第6.2.1.1节提供的信息算法中所述。

* 从接入单元 `AU(TS)` 中移除 `NI-MTAP` 和任何 `PACSI NAL` 单元。

* 接入单元 `AU(TS)` 按照与接入单元 `AU(TS)` 相关联的最高 `RTP` 会话中媒体时间戳值 `TS` 的出现顺序(由RTP序列号的顺序给出)传送给解码器。

  * 信息性说明：由于数据包丢失，有可能并非所有会话都有最高 `RTP` 会话中存在的媒体时间戳值 `TS` 的 `NAL` 单元。在这种情况下，算法可以：

    * a) 处理下一个在所有已接收的 `RTP` 会话中都有 `NAL` 单元存在的完整的接入单元

    * b) 考虑一个新的最高 `RTP` 会话，即该接入单元完整的最高 `RTP` 会话，并应用上述过程。  当接收到一个完整无误的接入单元，且该接入单元在所有的 `RTP` 会话中都包含 `NAL` 单元时，该算法可以返回到原来的最高 `RTP` 会话。

下面举一个实际的例子：

图6所示的例子是指三个`RTP`会话`A、B和C`，包含一个`SVC`比特流，作为3个源传输 在这个例子中，依赖性信令(在7.2.3节中描述)表明会话`A`是基础`RTP`会话，B是第一增强`RTP`会话并依赖于`A`，`C`是第二增强`RTP`会话并依赖于`A和B`，采用分层图片编码预测结构，其中会话`A`具有最低的帧率，会话`B`和`C`具有相同的序列但更高的帧率。

图中显示的是 `RTP` 数据包中包含的 `NAL` 单元，这些单元被存储在接收器的去抖动缓冲区中，用于会话解包。`NAL` 单元已经根据其 `RTP` 序列号顺序重新排序，如果在聚合包中，则根据其在聚合包中出现的顺序重新排序。图中显示了收到的`NAL`单元在会话中的解码顺序，以及相关的媒体(`NTP`)时间戳("TS[...]")。同一接入单元在会话内的`NAL`单元由"(.,.) "分组，并共享同一媒体时间戳`TS`，该时间戳显示在图的底部。请注意，时间戳不是按递增顺序排列的，因为在本例中，解码顺序与输出显示顺序不同。

该过程首先进行到与最高会话 `C` 中存在的第一个介质时间戳 `TS[1]` 相关联的 `NAL` 单元，并移除（或忽略）所有前面(按解码顺序)的 `NAL` 单元，直到 `RTP` 会话 `A、B和C` 的每一个去抖动缓冲区中带有 `TS[1]` 的 `NALU`。然后，从会话 `C` 开始，选择解码顺序中可用的第一个媒体时间戳(`TS[1]`)，并按照本备忘录第7.2.3节的要求，将从RTP会话A开始的 `NALU` 以及会话 `B` 和 `C` 按照 `RTP` 会话依赖性的顺序放置(在`TS[1]`的例子中：首先是会话 `B` ，然后是会话 `C` )到与媒体时间戳 `TS[1]` 相关联的访问单元 `AU(TS[1])` 中。  然后处理下一个媒体时间戳 `TS[3]`，按照出现在最高 `RTP` 会话 `C` 中的顺序，并重复上述过程。  注意，可能存在没有 `NALU` 存在的访问单元，例如，在最低的 `RTP` 会话 `A` 中(参见，例如，`TS[1]` )。对于 `TS[8]`，所有 `RTP` 会话中都存在 `NAL` 单元的第一个访问单元出现在缓冲区中。

    C: ------------(1,2)-(3,4)--(5)---(6)---(7,8)(9,10)-(11)--(12)----
         |     |     |     |     |     |      |    |     |      |
    B: -(1,2)-(3,4)-(5)---(6)--(7,8)-(9,10)-(11)-(12)--(13,14)(15,15)-
         |     |                 |     |                 |      |
    A: -------(1)---------------(2)---(3)---------------(4)----(5)----
    ---------------------------------------------------decoding order-->

    TS: [4]   [2]   [1]   [3]   [8]   [6]   [5]   [7]   [12]   [10]

    Key:
    A, B, C                - RTP sessions
    Integer values in "()" - NAL unit decoding order within RTP session
    "( )"                  - groups the NAL units of an access unit
                             in an RTP session
    "|"                    - indicates corresponding NAL units of the
                             same access unit AU(TS[..]) in the RTP
                             sessions
    Integer values in "[]" - media timestamp TS, sampling time
                             as derived, e.g., from NTP timestamp
                             associated with the access unit AU(TS[..]),
                             consisting of NAL units in the sessions
                             above each TS value.

图6.多源传输中的解码顺序恢复实例。

#### 6.2.1.1.  Informative Algorithm for NI-T Decoding Order Recovery within an Access Unit

在接入单元中，[H.264]规范（第7.4.1.2.3节和G.7.4.1.2.3节）限制了 `NAL` 单元的有效解码顺序。

这些约束使得仅根据每个会话中 `NAL` 单元的顺序、`NAL` 单元头和补充增强信息报文头，就可以为一个接入单元的 `NAL` 单元重建一个有效的解码顺序。

本节指定了一种信息算法，用于重建一个接入单元内 `NAL` 单元的有效解码顺序。其他的 `NAL` 单元顺序也可能是有效的，但是，任何符合要求的 `NAL` 单元顺序都将描述相同的视频流和辅助数据。

当然，一个实际的实现只需要 "接近" 这种重新排序的行为。特别是，在实现的解码过程中被丢弃的 `NAL` 单元不需要重新排序。

在该算法中，首先按访问单元内的 `NAL` 单元类型进行排序，顺序如下表12所规定的，但 `NAL` 单元类型 `14` 除外，该类型的 `NAL` 单元按表中所述的特殊处理。必要时，同一类型的 `NAL` 单元按该类型指定的顺序排序。

在本算法中，"会话顺序" 是指 `RTP` 会话中传输顺序所隐含的 `NAL` 单元的顺序。对于非交错和单 `NAL` 单元模式，这是 `RTP` 序列号顺序加上一个聚合单元中 `NAL` 单元的顺序。

表12.  接入单元内的 `NAL` 单元类型排序

     Type    Description / Comments
    -----------------------------------------------------------
      9      Access unit delimiter
 
      7      Sequence parameter set
 
      13     Sequence parameter set extension
 
      15     Subset sequence parameter set
 
      8      Picture parameter set
 
      16-18  Reserved
 
      6      Supplemental enhancement information (SEI)
             If an SEI message with a first payload of 0 (Buffering
             Period) is present, it must be the first SEI message.
 
             If SEI messages with a Scalable Nesting (30) payload and
             a nested payload of 0 (Buffering Period) are present,
             these then follow the first SEI message.  Such an SEI
             message with the all_layer_representations_in_au_flag
             equal to 1 is placed first, followed by any others,
             sorted in increasing order of DQId.
 
             All other SEI messages follow in any order.
 
      14     Prefix NAL unit in scalable extension
 
      1      Coded slice of a non-IDR picture
 
      5      Coded slice of an IDR picture
 
             NAL units of type 1 or 5 will be sent within only a
             single session for any given access unit.  They are
             placed in session order.  (Note: Any given access unit
             will contain only NAL units of type 1 or type 5, not
             both.)
 
             If NAL units of type 14 are present, every NAL unit of
             type 1 or 5 is prefixed by a NAL unit of type 14.  (Note:
             Within an access unit, every NAL unit of type 14 is
             identical, so correlation of type 14 NAL units with the
             other NAL units is not necessary.)
 
      12     Filler data
 
             The only restriction of filler data NAL units within an
             access unit is that they shall not precede the first VCL
             NAL unit with the same access unit.
 
      19     Coded slice of an auxiliary coded picture without
             partitioning
 
             These NAL units will be sent within only a single
             session for any given access unit, and are placed in
             session order.
 
       20    Coded slice in scalable extension
 
       21-23 Reserved
 
             Type 20 NAL units are placed in increasing order of DQId.
             Within each DQId value, they are placed in session order.
 
             (Note: SVC slices with a given DQId value will be sent
             within only a single session for any given access unit.)
 
             Type 21-23 NAL units are placed immediately following
             the non-reserved-type VCL NAL unit they follow in
             session order.
 
      10     End of sequence
 
      11     End of stream

### 6.2.2.  Decoding Order Recovery for the NI-C, NI-TC, and I-C Modes

当使用 `NI-C` 或 `I-C MST` 封装模式时，必须使用以下过程。  当使用 `NI-TC MST` 封装模式时，可以使用以下过程。

每个会话的 `RTP` 级接收处理输出的 `RTP` 数据包被放入再复用缓冲区。

建议将再复用缓冲区的大小(in bytes)设置为等于或大于接收端接收的最高 `RTP` 会话的 `sprop-remux-buf-req` 媒体类型参数的值。

计算并存储每个 `NAL` 单元的 `CS-DON` 值。

* 信息性说明：一个 `NAL` 单元的 `CS-DON` 值可能依赖于另一个数据包中携带的信息，而不是包含在该 `NAL` 单元的数据包中，例如，当单 `NAL` 单元数据包中包含的非 `PACSI NAL` 单元需要推导 `CS-DON` 值时，就会发生这种情况，因为单 `NAL` 单元数据包本身不包含 `CS-DON` 信息。在这种情况下，当一个 `NAL` 单元没有收到包含所需 `CS-DON` 信息的数据包时，这个 `NAL` 单元必须被接收端丢弃，因为它不能以正确的顺序被送入解码器。当可选的介质类型参数 `sprop-mst-csdon-always-present` 等于1时，不存在这种依赖性，即任何特定 `NAL` 单元的 `CS-DON` 值可以仅根据包含 `NAL` 单元的数据包中的信息得出，因此，接收方不需要丢弃任何收到的 `NAL` 单元。

下面借助以下函数和常量对接收端的操作进行描述。

* `AbsDON` 函数在[RFC6184]第8.1节中规定。

* `don_diff` 函数在[RFC6184]第5.5节中规定。

* 常量 `N` 是最高 `RTP` 会话的可选 `sprop-mst-remux-buf-size` 媒体类型参数的值，按 `1` 递增。

初始缓冲会一直持续到满足以下条件之一。

* 再复用缓冲区中有 `N` 个或多个 `VCL NAL` 单元。

* 如果最高 `RTP` 会话的 `sprop-mst-max-don-diff` 存在，`don_diff(m,n)` 大于最高 `RTP` 会话的 `sprop-mst-max-don-diff` 的值，其中 `n` 对应于接收到的 `NAL` 单元中 `AbsDON` 值最大的 `NAL` 单元，`m` 对应于接收到的 `NAL` 单元中 `AbsDON` 值最小的NAL单元。

* 初始缓冲的持续时间等于或大于最高 `RTP` 会话的可选 `sprop-remux-init-buf-time` 媒体类型参数值。

从再复用缓冲区中取出的 `NAL` 单元确定如下。

* 如果再复用缓冲区至少包含 `N` 个 `VCL NAL` 单元，则从再复用缓冲区中取出 `NAL` 单元，并按下面的要求顺序传递给解码器，直到缓冲区中包含 `N-1` 个 `VCL NAL` 单元。

* 如果存在最高 `RTP` 会话的 `sprop-mst-max-don-diff`，则所有 `don_diff(m,n)` 大于最高 `RTP` 会话的 `sprop-max-don-diff` 的 `NAL` 单元 `m` 将从再复用缓冲区中移除，并按下面规定的顺序传递给解码器。这里，`n` 对应于再复用缓冲区中的 `NAL` 单元中具有 `AbsDON` 的最大值的 `NAL` 单元。

`NAL` 单元传递给解码器的顺序规定如下。

* 设 `PDON` 为一个变量，在 `RTP` 会话开始时初始化为 `0`。

* 对于每个与 `CS-DON` 值相关联的 `NAL` 单元，`CS-DON` 的距离计算如下：如果 `NAL` 单元的 `CS-DON` 值大于 `PDON` 值，则 `CS-DON` 距离等于 `CS-DON-PDON` 。  如果NAL单元的CS-DON值大于PDON值，则CS-DON距离等于CS-DON-PDON，否则，CS-DON距离等于65535-PDON+CS-DON+1。  否则，CS-DON距离等于65535-PDON+CS-DON+1。

* NAL单元按CS-DON距离的递增顺序传递给解码器。  如果几个NAL单元的CS-DON距离值相同，它们可以以任何顺序传递给解码器。

* 当所需的NAL单元数被传递给解码器时，PDON的值被设置为传递给解码器的最后一个NAL单元的CS-DON的值。

# 7.  Payload Format Parameters
略

# 8.  Security Considerations

解码者必须谨慎处理保留的 `NAL` 单元类型和保留的 `SEI` 消息，特别是当它们包含活动元素时，并且必须将它们的适用范围限制在明确包含这些元素的流上。最安全的方法是直接丢弃这些 `NAL` 单元和 `SEI` 消息。

当完整性保护应用于一个流时，必须注意被传输的流可能是可扩展的；因此，应答者可能只能访问整个流的一部分。

端到端安全认证、完整性或保密性保护将防止 `MANEs` 执行媒体感知操作，而是只能丢弃完整的数据包。 而在保密性保护的情况下，`MANEs` 甚至会被阻止以媒体感知的方式执行丢弃数据包的操作。为了允许任何 `MANEs` 执行其操作，它将被要求成为一个包含在安全上下文建立中的可信实体。这既适用于媒体路径，也适用于 `RTCP` 路径，如果 `RTCP` 包需要重写的话。

# 9.  Congestion Control

降低会话比特率可以通过以下一种或多种方式实现。

a) 在 `DID` 字段确定的最高层内，删除 `QID` 高于某一数值的任何NAL单元。

b) 删除所有 `TID` 高于某一数值的 `NAL` 单元。

c) 删除所有与 `DID` 高于某一数值的 `NAL` 单位。

* 信息性说明：为了保持 `SVC` 流的一致性，需要删除所有与 `DID` 高于整个流中某个值的编码片 相关的 `NAL` 单元。

d) 利用 `PRID` 字段表明 `NAL` 单位的相对重要性，并删除所有与 `PRID` 高于某一数值相关的的 `NAL` 单位。请注意，`PRID` 的使用是针对具体应用的。

(e) 根据特定于应用的规则删除 `NAL` 单元或整个数据包。其结果将取决于所使用的特定编码结构以及任何额外的特定应用功能(例如，在接收解码器上进行的隐藏)。一般来说，这将导致接收到一个不符合要求的比特流，因此解码器的行为不是由[H.264]规定的。因此，如果特定的解码器实现没有采取适当的措施来响应拥塞控制，那么在解码输出中可能会出现重大的伪影。

* 信息性说明：上面的讨论集中在 `NAL` 单位而不是数据包上，主要是因为那是使用者可以有意义地操作可扩展比特流的层次。当使用聚合数据包时，将 `NAL` 单元映射到 `RTP` 数据包是相当灵活的。  根据拥塞控制算法的性质，拥塞测量的 "维度"(包数或比特率)和对它的反应(减少包数或比特率或两者)可以相应调整。

所有上述手段对 `RTP` 发送者都是可用的，不管该发送者是位于发送端点还是位于基于混合器的 `MANEs`中。

当采用基于转换器的 `MANEs` 时，那么 `MANE` 只能在 `MANE` 的出站路径上操纵会话，使感知到的端到端拥塞回落在允许的包络范围内。与所有转换器一样，在这种情况下，`MANE` 需要重写 `RTCP RR`，以反映它对会话进行的操纵。

* 信息性说明：应用程序也可以实现其他拥塞控制机制，如[RFC5775]和[Yan]中描述的机制。

# 10.  IANA Considerations

本备忘录第7.1节规定的新媒体类型已在IANA注册。

# 11.  Informative Appendix: Application Examples

## 11.1.  Introduction

可扩展的视频编码是一个概念，从MPEG-2[MPEG2]开始就有了，最早可以追溯到1993年。然而，它从来没有得到广泛的接受，也许部分原因是应用没有以标准化所设想的形式实现。

`ISO/IEC MPEG` 和 `ITU-T VCEG` 分别对 `SVC` 项目进行了需求分析。`MPEG` 和 `VCEG` 的需求文件分别为 [JVT-N026]和[JVT-N027]。

下面介绍四种主要的应用场景，作者认为这些应用场景是相关的，并且可以用本规范来实现。

## 11.2.  Layered Multicast

这种广为人知的分层编码使用形式 [McCanne] 意味着所有层都在自己的 `RTP` 包流中单独传达，每个层都在自己的 `RTP` 会话中进行，使用 `IP` (组播)地址和端口号作为单个解复用点。接收器通过订阅 `IP` 组播来 "调谐 "到各层，通常使用 `IGMP[IGMP]`。根据应用场景的不同，当不需要在层的子集内进行更精细的操作点时，也可以在一个 `RTP` 会话中传递多个层。

分层组播具有简单和易于实现的巨大优势。但是，它也有一个巨大缺点，即需要利用许多不同的传输地址的。虽然笔者认为这对于一个经过专业维护的内容服务器来说并不是一个大问题，但接收客户端的终端需要打开许多端口到防火墙中的 `IP` 多播地址。从防火墙和网络地址转换(`NAT`)的角度来看，这是一个不切实际的问题。此外，即使在今天，`IP` 组播也没有像许多人希望的那样被广泛部署。

作者认为分层组播是一个重要的应用场景，原因如下。首先，它是很好理解的，而且其实现限制也是众所周知的。其次，在当前的互联网环境之下，可能会有大规模的IP网络在未来有希望采用分层组播。一个可能的例子可以是为各种移动电视业务，例如 `3GPP(MBMS)[MBMS]` 和 `DVB(DVB-H)[DVB-H]` 正在开发的移动电视业务，将内容创建和核心网分发结合起来。

## 11.3.  Streaming

在这种情况下，流媒体服务器有一个给定内容的 `SVC` 编码层的存储库。在流媒体播放时，根据客户端的能力、连接性和拥堵情况，流媒体服务器生成并提供可升级的流媒体。可能通过单播和多播服务都。  同时，流媒体服务器可以使用相同的存储层库来组成不同的流媒体（具有不同的层集），以满足其他受众的需求。

由于每个端点只接收一个 `SVC RTP` 会话，防火墙打洞的数量可以优化为一个。

这种方案与直截了当的组播的主要区别在于流媒体服务器的架构和要求，因此不在 `IETF` 标准化的范围内。然而，关于为什么这样的流媒体服务器设计是有意义的可以提出一些令人信服的论点。一个可能的论点是与存储空间和通道带宽有关；另一个是无需转码的带宽适应性 -- 在拥塞控制网络中是一个相当大的优势。  当流媒体服务器了解到拥堵时，它可以在组成分层流时选择较少的层数来降低发送比特率，祥见第9节。 `SVC` 的设计是以相当大的动态范围优雅地支持带宽上升和下降。虽然理论上，转码步骤可以实现类似的动态范围，但计算需求不切实际地高，而且视频质量通常会降低--因此，很少有（如果有的话）流媒体服务器实现完全的转码。

## 11.4.  Video conferencing (Unicast to MANE, Unicast to Endpoints)

视频会议传统上依赖多点控制单元（`MCU`）。这些单元以星型配置连接端点，其工作原理如下：编码视频从每个终端传输到 `MCU`，在那里它被解码、缩放和合成，形成输出帧，然后重新编码并传输到终端。在支持个性化布局的系统中(每个用户被允许选择他的屏幕的布局)，合成和编码过程是针对每个接收端点进行的。即使没有个性化布局，速率匹配仍然需要在 `MCU` 上为每个端点单独执行编码过程。因此，`MCU` 具有相当的复杂性，并引入了显著的延迟。级联编码也会降低视频质量，特别是对于多点连接，由于端到端延迟非常高，交互式通信非常麻烦[G.114]。一个比较简单的架构是切换 `MCU` ，其中一个传入的视频流被重定向到接收端点。显然，一次只能看到一个用户，无法进行速率匹配，从而迫使所有传输端点以 `MCU` 到端点连接中可用的最低比特率进行传输。

在可扩展的视频编码中，`MCU` 可以用应用级路由器( `ALR` )来代替：这个单元只需选择哪些传入的数据包应该传送到哪个接收端点[Eleft]。在这样的系统中，每个端点都会对传入的视频流执行它自己的的合成。例如，假设作为使用两层空间可扩展性的系统，个性化布局相当于指示 `ALR` 只将相应分辨率所需的数据包发送到特定的端点。同样，在 `ALR` 为特定端点进行速率匹配，可以通过选择一个适当的传入视频子以传输到特定的端点来实现。因此，个性化布局和速率匹配成为路由决策，不再需要有信令进行处理。请注意，可扩展性还允许参与者享受其链路所提供的最佳视频质量，即用户不必再被迫在最弱的端点所支持的质量下操作。最重要的是，`ALR` 对端到端延迟的影响微不足道，通常比 `MCU` 少一个数量级。这使得它有可能与甚至非常多的参与者进行完全交互式的多点会议，在容错能力方面也有显著的优势，事实上，这里的容错能力也可以增加近一个数量级(例如，使用不平等的错误保护)。最后，`ALR` 的极低延迟允许这些系统级联，在系统设计和部署方面具有显著的优势，传统的 `MCU` 是不可能级联的，因为即使是单个 `MCU` 也会带来非常高的延迟。

可扩展的视频编码使视频会议系统实现了非常重大的模式转变，使视频通信系统（特别是驻扎在网络中的服务器）的复杂性与其他类型的网络应用变的一样了。

## 11.5.  Mobile TV (Multicast to MANE, Unicast to Endpoint)

这种方案比较复杂，旨在优化核心网络中的网络流量，同时仍然只需要在终端的防火墙上打一个洞。其关键应用之一是移动电视市场。

考虑一个大型的私有 `IP` 网络，例如，第三代合作伙伴计划（`3GPP`）的核心网络。可以假设这个核心网络中的流媒体服务器是专业维护的。假设这些服务器可以有许多端口开放到网络上，并且分层组播是一个真实可用的选择，那么，流媒体服务器就可以组播 `SVC` 可扩展层，而不是在不同的比特率下联播推送相同内容的不同表现。

还可以考虑许多不同类别的终端。这些终端中的一些可能缺乏处理能力或显示尺寸，无法对所有层级进行精确解码；而另一些则可能具备这些能力，一些终端的用户可能不希望为高质量付费，而对基本服务感到满意，这种服务可能更便宜甚至免费，而其他用户则愿意为高质量付费。最后，一些连接用户可能存在带宽问题，他们无法接收到他们希望接收的带宽--无论是通过拥堵、连接、服务质量的变化，还是其他任何原因。还有，所有这些用户都有一个共同点，那就是他们不想暴露太多，因此防火墙打洞的数量要少。

这种情况最好的处理方法是引入靠近核心网边缘的中间盒，中间盒接收分层组播流，并根据所连接的端点的需求，组成单一的 `SVC` 可扩展比特流。这些中间盒在本规范中称为 `MANE` 。  在实践中，作者认为 `MANE` 是移动网络基站的一部分(或至少在物理上和拓扑上接近)，其中所有的信令和媒体流量必然是在同一物理链路上复用的。

`MANE` 必然需要是相当复杂的设备，它们当然需要理解信令，例如，将 `RTP` 头中的 `payload` 类型八位组与 `SVC` 有效载荷类型联系起来。

一个 `MANE` 可以聚合多个 `RTP` 流，可能来自多个 `RTP` 会话，从而减少终端所需的防火墙打洞数量，或者可以通过利用本备忘录的聚合和分片机制，将输出的 `RTP` 流优化到输出路径的 `MTU` 大小。这种类型的 `MANE` 在概念上很容易实现，并且可以提供强大的功能，主要是因为它必然可以 "看到" 有效载荷(包括 `RTP` 有效载荷头)，利用其中丰富的分层信息，并对其进行操作。

`MANE` 还可以进行流稀释，以遵守第9节讨论的拥塞控制原则。虽然这种 `MANE` 的前向(媒体)信道的实现看起来比较简单，但由于需要重写 `RTCP RRs`，即使是这样的 `MANE` 也是一个复杂的设备。

虽然如上所述，`MANE` 的任何一种情况下的实施复杂性都相当高，但计算需求相对较低。

# 13.  References

## 13.1.  Normative References

   [H.264]    ITU-T Recommendation H.264, "Advanced video coding for generic audiovisual services", March 2010.

   [RFC6184]  Wang, Y.-K., Even, R., Kristensen, T., and R. Jesup, "RTP Payload Format for H.264 Video", RFC 6184, May 2011.

   [ISO/IEC14496-10] ISO/IEC International Standard 14496-10:2005.

   [RFC2119]  Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, March 1997.

   [RFC3264]  Rosenberg, J. and H. Schulzrinne, "An Offer/Answer Model with Session Description Protocol (SDP)", RFC 3264, June 2002.

   [RFC3550]  Schulzrinne, H., Casner, S., Frederick, R., and V. Jacobson, "RTP: A Transport Protocol for Real-Time Applications", STD 64, RFC 3550, July 2003.

   [RFC4288]  Freed, N. and J. Klensin, "Media Type Specifications and Registration Procedures", BCP 13, RFC 4288, December 2005.

   [RFC4566]  Handley, M., Jacobson, V., and C. Perkins, "SDP: Session Description Protocol", RFC 4566, July 2006.

   [RFC4648]  Josefsson, S., "The Base16, Base32, and Base64 Data Encodings", RFC 4648, October 2006.

   [RFC5576]  Lennox, J., Ott, J., and T. Schierl, "Source-Specific Media Attributes in the Session Description Protocol (SDP)", RFC 5576, June 2009.

   [RFC5583]  Schierl, T. and S. Wenger, "Signaling Media Decoding Dependency in the Session Description Protocol (SDP)", RFC 5583, July 2009.

   [RFC6051]  Perkins, C. and T. Schierl, "Rapid Synchronisation of RTP Flows", RFC 6051, November 2010.

13.2.  Informative References

   [DVB-H]    DVB - Digital Video Broadcasting (DVB); DVB-H Implementation Guidelines, ETSI TR 102 377, 2005.

   [Eleft]    Eleftheriadis, A., R. Civanlar, and O. Shapiro, "Multipoint Videoconferencing with Scalable Video Coding", Journal of Zhejiang University SCIENCE A, Vol. 7, Nr. 5, April 2006, pp. 696-705. (Proceedings of the Packet Video 2006 Workshop.)

   [G.114]    ITU-T Rec. G.114, "One-way transmission time", May 2003.

   [H.241]    ITU-T Rec. H.241, "Extended video procedures and control signals for H.300-series terminals", May 2006.

   [IGMP]     Cain, B., Deering, S., Kouvelas, I., Fenner, B., and A. Thyagarajan, "Internet Group Management Protocol, Version 3", RFC 3376, October 2002.

   [JVT-N026] Ohm J.-R., Koenen, R., and Chiariglione, L. (ed.), "SVC requirements specified by MPEG (ISO/IEC JTC1 SC29 WG11)", JVT-N026, available from http://ftp3.itu.ch/av-arch/ jvt-site/2005_01_HongKong/JVT-N026.doc, Hong Kong, China,
              January 2005.

   [JVT-N027] Sullivan, G. and Wiegand, T. (ed.), "SVC requirements specified by VCEG (ITU-T SG16 Q.6)", JVT-N027, available from http://ftp3.itu.int/av-arch/ jvt-site/2005_01_HongKong/JVT-N027.doc, Hong Kong, China, January 2005.

   [McCanne]  McCanne, S., Jacobson, V., and Vetterli, M., "Receiver- driven layered multicast", in Proc. of ACM SIGCOMM'96, pages 117-130, Stanford, CA, August 1996.

   [MBMS]     3GPP - Technical Specification Group Services and System Aspects; Multimedia Broadcast/Multicast Service (MBMS); Protocols and codecs (Release 6), December 2005.

   [MPEG2]    ISO/IEC International Standard 13818-2:1993.

   [RFC2326]  Schulzrinne, H., Rao, A., and R. Lanphier, "Real Time Streaming Protocol (RTSP)", RFC 2326, April 1998.

   [RFC2974]  Handley, M., Perkins, C., and E. Whelan, "Session Announcement Protocol", RFC 2974, October 2000.

   [RFC5117]  Westerlund, M. and S. Wenger, "RTP Topologies", RFC 5117, January 2008.

   [RFC5775]  Luby, M., Watson, M., and L. Vicisano, "Asynchronous Layered Coding (ALC) Protocol Instantiation", RFC 5775, April 2010.

   [Yan]      Yan, J., Katrinis, K., May, M., and Plattner, R., "Media- and TCP-friendly congestion control for scalable video streams", in IEEE Trans. Multimedia, pages 196-206, April 2006.