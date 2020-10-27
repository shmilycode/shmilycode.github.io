# SDP: Session Description Protocol

RFC3266

Copyright Notice
   Copyright (C) The Internet Society (2006).

# Abstract

本备忘录定义了会话描述协议（SDP）。SDP旨在描述多媒体会话，用于会话公告、会话邀请和其他形式的多媒体会话启动。

<!-- TOC -->

- [SDP: Session Description Protocol](#sdp-session-description-protocol)
- [Abstract](#abstract)
- [1. 引言](#1-引言)
- [2. 术语表](#2-术语表)
- [3. SDP使用实例](#3-sdp使用实例)
  - [3.1.  会话初始化](#31-会话初始化)
  - [3.2.  流媒体](#32-流媒体)
  - [3.3.  电子邮件和万维网](#33-电子邮件和万维网)
  - [3.4.  多播会话公告](#34-多播会话公告)
- [4.  要求与建议](#4-要求与建议)
  - [4.1.  介质和传输信息](#41-介质和传输信息)
  - [4.2.  时间信息](#42-时间信息)
  - [4.3. 私有会话](#43-私有会话)
  - [4.4.  获取关于会话的进一步信息](#44-获取关于会话的进一步信息)
  - [4.5.  分类](#45-分类)
  - [4.6.  国际化](#46-国际化)
- [5. SDP规范](#5-sdp规范)
  - [5.1.  协议版本("v=")](#51-协议版本v)
  - [5.2.  Origin("o=")](#52-origino)
  - [5.3.  会话名称("s=")](#53-会话名称s)
  - [5.4.  会话信息("i=")](#54-会话信息i)
  - [5.5.  URI ("u=")](#55-uri-u)
  - [5.6.  电子邮件地址和电话号码("e="和 "p=")](#56-电子邮件地址和电话号码e和-p)
  - [5.7.  连接数据("c=")](#57-连接数据c)
  - [5.8.  带宽("b=")](#58-带宽b)
  - [5.9.  定时("t=")](#59-定时t)
  - [5.10.  重复时间("r=")](#510-重复时间r)
  - [5.12.  加密密钥("k=")](#512-加密密钥k)
  - [5.13.  属性("a=")](#513-属性a)
  - [5.14.  媒体描述("m=")](#514-媒体描述m)
- [6.SDP属性](#6sdp属性)
- [7. 安全因素考虑](#7-安全因素考虑)
- [8.  IANA的考虑](#8-iana的考虑)
  - [8.1.  "application/sdp "媒体类型](#81-applicationsdp-媒体类型)
  - [8.2.  注册参数](#82-注册参数)
    - [8.2.1. 媒体类型("media")](#821-媒体类型media)
    - [8.2.2.  传输协议("proto")](#822-传输协议proto)
    - [8.2.3.  媒体格式("fmt")](#823-媒体格式fmt)
    - [8.2.4.  属性名称("att-field")](#824-属性名称att-field)
    - [8.2.5. 带宽规格（"bwtype"）](#825-带宽规格bwtype)
    - [8.2.6.  新的网络类型（"nettype"）](#826-新的网络类型nettype)
    - [8.2.7.  新的地址类型("addrtype")](#827-新的地址类型addrtype)
    - [8.2.8.  注册程序](#828-注册程序)
  - [8.3.  加密密钥访问方法](#83-加密密钥访问方法)
- [9. SDP语法](#9-sdp语法)
- [12. 参考](#12-参考)
  - [12.1.  Normative References](#121-normative-references)
  - [12.2.  Informative References](#122-informative-references)

<!-- /TOC -->

# 1. 引言

当发起多媒体电话会议、IP语音通话、流媒体视频或其他会话时，需要向与会者传达媒体细节、传输地址和其他会话描述元数据。

SDP为这种信息提供了一种标准的表示方式，而不管这种信息是如何传输的。SDP纯粹是一种会话描述格式 -- 它并不关心传输协议是什么，它的目的只是可以被不同的传输协议适当地传输，包括会话公告协议（Session Announcement Protocol）[14]、会话初始化协议（Session Initiation Protocol）[15]、实时流协议（Real Time Streaming Protocol）[16]、使用MIME扩展的电子邮件和超文本传输协议。

SDP的设计目的是通用性，因此它可以在各种网络环境和应用中被广泛的使用，但它不打算支持会话内容或媒体编码的协商：这被视为超出了会话描述的范围。

本备忘录废除了RFC 2327[6]和RFC 3266[10]。第10节概述了本备忘录中引入的变化。

# 2. 术语表

本文件中使用了下列术语，这些术语在本文件中具有特定含义：

* 会议：多媒体会议是由两个或两个以上的通信用户以及他们所使用的软件组成。

* 会话：多媒体会话是指一组多媒体发送者和接收者，以及从发送者流向接收者的数据流。多媒体会议就是多媒体会话的一个例子。

* 会话描述：一个定义明确的格式，用于传达足够的信息，以便发现和参与多媒体会议。

# 3. SDP使用实例

## 3.1.  会话初始化

会话初始化协议(SIP)[15]是一个应用层控制协议，用于创建、修改和终止会话，如互联网多媒体会议、互联网电话和多媒体分发。用于创建会话的SIP消息带有会话描述符，允许参与者协商并同意一组兼容的媒体类型。这些会话描述符通常使用SDP格式化，当与SIP一起使用时，offer/answer模式[17]为使用SDP进行协商提供了一个有限的框架。

## 3.2.  流媒体

实时流协议(RTSP)[16]，是一种应用级协议，用于控制具有实时属性的数据的传输。RTSP提供了一个可扩展的框架，以实现实时数据（如音频和视频）的控制、按需传输。RTSP客户端和服务器协商出一套合适的媒体传输参数，并部分使用SDP语法来描述这些参数。

## 3.3.  电子邮件和万维网

传递会话描述符的替代手段包括电子邮件和万维网。如果通过电子邮件和万维网的方式传播，需要使用了 "application/sdp "这一媒介类型。这使得可以从 WWW 客户端或邮件阅读器以标准方式自动启动参与会话的应用程序。

请注意，仅通过电子邮件或WWW发布的多播会话公告，并不能保证会话公告的接收者一定能接收该会话，因为多播会话可能在范围上受到限制，而在这个范围之外也可以访问WWW服务或接收电子邮件。

## 3.4.  多播会话公告

为了协助多播多媒体会议和其他多播会议的公告，并将相关的会议设置信息传达给潜在的参与者，可以使用分布式会议目录。这种会话目录的一个实例是定期向一个著名的多播组发送包含会话描述符的数据包。这些公告由其他会话目录接收，这样潜在的远程参与者可以使用会话描述符来启动参与会话所需的工具。

用于实现这种分布式目录的一个协议是会话公告协议（SAP）[14]。SDP为这种会话公告提供了推荐的会话描述格式。

# 4.  要求与建议

SDP的目的是在多媒体会议中传递有关媒体流的信息，使会议描述的接收者能够参与会议。尽管SDP具有足够的通用性，可以描述其他除了互联网以外的网络环境中的会议，但它主要还是为了在internet网络中使用。媒体流可以是多对多，且不需要持续保持活跃状态。

迄今为止，互联网上基于多播的会话与其他许多形式的会议不同，因为任何接收流量的人都可以加入会话（除非会话流量是加密的）。在这种环境下，SDP主要服务于两个目的，首先它是一种沟通会话存在的手段，其次它也是一种传递足够信息以加入和参与会话的手段。在单播环境中，只有后一个目的可能是相关的。

一个SDP会话描述包括以下内容：

* 会话名称和目的 

* 会话处于活动状态的时间 

* 组成会话的媒体

* 接收这些媒体所需的信息（地址、端口、格式等）

由于参与会话所需的资源可能是有限的，因此也需要一些附加信息。

* 关于会话使用的带宽相关的信息

* 会议负责人的联系信息

一般来说，SDP必须传递足够的信息，使应用程序能够加入会话(可能的加密密钥除外)，并向任何可能需要知道的非参与者宣布要使用的资源（当SDP与多播会话公告协议一起使用时，后一种功能非常有用）

## 4.1.  介质和传输信息

SDP会话描述包括以下媒体信息。

* 媒体的类型（视频、音频等）

* 传输协议（RTP/UDP/IP、H.320等）

* 媒体的格式（H.261视频、MPEG视频等）

除了媒体格式和传输协议，SDP还传递地址和端口的详细信息。对于一个IP组播会话，包括如下内容。

* 媒体的组播组地址

* 媒体的传输端口

这个地址和端口是组播流的目的地址和目的端口，无论是发送、接收，还是两者兼有。

对于单播IP会话，传达了以下内容。

* 媒体的远程地址

* 媒体的远程传输端口

这个地址和端口的语义取决于定义的媒体和传输协议。默认情况下，这应该是发送数据的远程地址和远程端口。一些媒体类型可能会重新定义这种行为，但并不推荐这么做，因为它使实现变得复杂（包括必须将地址解析为开放网络地址转换（NAT）或防火墙孔的中间盒）。

## 4.2.  时间信息

会话在时间上可以是有界的，也可以是无界的。无论是否是有界的，都可能只在特定的时间活动。SDP可以用于传达：

* 会话开始和停止时间范围的任意列表

* 每个范围的重复时间，如 "每周三上午10点一小时"。

无论当地时区或夏令时是怎么样的，这些时间信息都是全球统一的（见5.9节）。

## 4.3. 私有会话

用户可以创建公有会话和私有会话，SDP本身并不区分这两者，私有会话通常是通过在分发过程中加密会话描述符来进行传递，加密方式的细节取决于用于传输SDP的机制；目前已经为使用SAP[14]和SIP[15]传输的SDP定义了机制，将来还可能定义其他机制。

如果一个会议公告是私有的，那么就可以使用该私有公告来传递会议中每个媒体解密所需的加密密钥，包括足够的信息来了解每个媒体使用的加密方案。

## 4.4.  获取关于会话的进一步信息 

会话描述应提供足够的信息，以便决定是否参加会话。SDP可以通过包含统一资源标识符(URI)形式的附加指针的方式，以用于获取关于会话的更多信息。

## 4.5.  分类

当SAP或任何其他广告机制分发许多会话描述符时，可能需要将感兴趣的会话公告从不感兴趣的会话公告中过滤出来。SDP为能够自动进行的会话提供了一种分类机制（"a=cat:" 属性；见第 6 节）。

## 4.6.  国际化

建议使用UTF-8编码中的ISO 10646字符集[5]，以允许表示许多不同的语言。然而，为了有助于进行紧凑的表示，SDP也允许在需要的时候使用其他字符集，如ISO 8859-1。国际化只适用于自由文本字段（会话名称和背景信息），而不是整个SDP。

# 5. SDP规范

SDP会话描述由媒体类型 "application/sdp "表示（见第8节）。

SDP会话描述使用纯文本格式，使用UTF-8编码的ISO 10646字符集。SDP字段名和属性名只使用UTF-8的US-ASCII子集，但文本字段和属性值可以使用完整的ISO 10646字符集。使用完整UTF-8字符集的字段和属性值从不直接比较，因此不需要UTF-8标准化。选择文本形式，而不是ASN.1或XDR等二进制编码，是为了增强可移植性，使各种传输方式得以使用，并允许使用灵活的、基于文本的工具包来生成和处理会话描述。 然而，由于SDP可以在会话描述的最大允许大小受到限制的环境中使用，因此编码被有意压缩。此外，由于公告可能会通过非常不可靠的方式传输或被中间缓存服务器损坏，因此编码设计了严格的顺序和格式化规则，以便大多数错误会导致会话公告变的畸形，从而可以很容易地被检测到并丢弃。这也允许快速丢弃接收器没有正确密钥的加密会话公告。

SDP会话描述由若干行文字组成，其形式如下：

    <type>=<value>

其中 `<type>` 必须是一个大小写敏感的字符，`<value>` 是结构化文本，其格式取决于`  `<type>`。一般来说，`<value>`是由一个空格字符或自由格式字符串分隔的若干字段，并且是大小写敏感的，除非特定字段另有定义。在 "=" 号的两边一定不能使用空格。

一个SDP会话描述的组成是，一个会话级的部分，后面是零个或多个媒体级的部分。会话级部分以 "v=" 行开始，一直到第一个媒体级部分。每个媒体级部分以 "m=" 行开始，并持续到下一个媒体级部分或整个会话描述的结束。一般来说，会话级的值是所有媒体的默认值，除非被等效的媒体级的值覆盖。

在每个描述中，有些行是REQUIRED，有些是OPTIONAL，但所有的行都必须按照这里给出的顺序出现（固定的顺序极大地提高了错误检测能力，并允许一个简单的解析器）。

会话描述

    v= (协议版本)
    o= (发起人和会话标识符)
    s= (会话名称)
    i=* (会话信息)
    u=* (描述的URI)
    e=* (电子邮件地址)
    p=* (电话号码)
    c=* (连接信息--如果包含在所有媒体中则不需要)
    b=* (零或多个带宽信息行)
    一个或多个时间描述("t="和 "r="行。 见下文)
    z=*(时区调整)
    k=*(加密密钥)
    a=*(零个或多个会话属性行)
    零个或多个媒体描述

时间描述

    t=(会话活动时间)
    r=*(零个或多个重复次数)

媒体描述，如果存在的话

    m= (媒体名称和传输地址)
    i=* (媒体标题)
    c=* (连接信息--如果包含在会话级别，则为可选)
    b=* (零或更多带宽信息行)
    k=* (加密密钥)
    a=* (零或更多媒体属性行)

类型字母的集合故意设计的很小，而且不打算让其可扩展 -- 一个SDP解析器必须完全忽略包含它不理解的类型字母的任何会话描述。属性机制（下文描述的 "a="）是扩展 SDP 并使其适应特定应用或媒体的主要手段。一些属性（本备忘录第 6 节中列出的属性）有明确的含义，但其他属性可以根据应用程序、媒体或会话的具体情况添加。SDP 解析器必须忽略任何它不理解的属性。

SDP会话描述可能包含需要引用到"u="、"k="和 "a="行的URI，在某些情况下，这些URI可能会被引用，使得会话描述不再完全独立。

会话级部分的连接("c=")和属性("a=")信息适用于该会话的所有媒体，除非被媒体描述中的连接信息或同名属性所覆盖。例如，在下面的例子中，每个媒体的行为都被赋予了一个 "recvonly" 属性。

一个SDP描述的例子是：

    v=0
    o=jdoe 2890844526 2890842807 IN IP4 10.47.16.5
    s=SDP Seminar
    i=A Seminar on the session description protocol
    u=http://www.example.com/seminars/sdp.pdf
    e=j.doe@example.com (Jane Doe)
    c=IN IP4 224.2.17.12/127
    t=2873397496 2873404696
    a=recvonly
    m=audio 49170 RTP/AVP 0
    m=video 51372 RTP/AVP 99
    a=rtpmap:99 h263-1998/90000

会话名称和信息等文本字段是八位字符串，可以包含任何八位数，但0x00(Nul)、0x0a(ASCII新行)和0x0d(ASCII回车)除外。序列 CRLF (0x0d0a) 用于结束记录，尽管解析器还应该容忍并接受以单个换行符结束的记录。如果 "a=charset "属性不存在，这些八位字符串必须被解释为包含UTF-8编码的ISO-10646字符（"a=charset "属性的存在可能会迫使一些字段被赋予不同的解释）。

一个会话描述可以在 "o="、"u="、"e="、"c="和 "a="行中包含域名。任何在SDP中使用的域名必须符合[1]、[2]的规定。国际化域名(IDNs)必须使用[11]中定义的ASCII兼容编码(ACE)形式来表示，不得直接使用UTF-8或其他编码来表示(这一要求是为了与RFC 2327和其他SDP相关标准兼容，这些标准在国际化域名发展之前就已存在)。

## 5.1.  协议版本("v=")

    v=0 

"v="字段给出了会话描述协议的版本，本备忘录定义了版本0，没有小版本号。

## 5.2.  Origin("o=")

    o=<username> <sess-id> <sess-version> <nettype> <addrtype><unicast-address>
    
"o="字段给出了会话的发起人(她的用户名和用户主机的地址)加上会话标识和版本号。


`<username>` 是用户在始发主机上的登录名，如果始发主机不支持用户ID的概念，则为"-"，`<username>` 不能包含空格。

`<sess-id>` 是一个数字字符串，这样 `<username>、<sess-id>、<nettype>、<addrtype>和<unicast-address>` 的元组就形成了会话的全球唯一标识符。`<sess-id>` 的分配方法由创建工具决定，但有人建议使用网络时间协议(NTP)格式的时间戳来确保唯一性[13]。

`<sess-version>` 是这个会话描述的版本号。它的使用由创建工具决定，只要当对会话数据进行修改时，`<sess-version>`就会增加。同样，建议使用 NTP 格式的时间戳。

`<nettype>`是一个文本字符串，表示网络的类型。最初 "IN "被定义为具有 "Internet "的含义，但将来可以注册其他值（见第8节）。

`<addrtype>`是一个文本字符串，给出后面地址的类型。最初定义的是 "IP4 "和 "IP6"，但将来可以注册其他值（见第8节）。

`<unicast-address>`是创建会话的机器的地址。对于地址类型为IP4的机器，这是该机器的完全限定域名或该机器IP版本4地址的点阵十进制表示。对于IP6的地址类型，这是机器的完全限定域名或机器的IP版本6地址的压缩文本表示。对于IP4和IP6，完全限定的域名是应该给出的形式，除非无法获得，在这种情况下，可以用全球唯一的地址来代替。本地 IP 地址不得用于任何 SDP 描述可能会离开该地址有意义的范围的上下文中（例如，本地地址不得包含在可能会离开范围的应用级引用中）。

一般来说，"o="字段是这个版本的会话描述的全球唯一标识符，与版本外的其他子字段共同标识该会话，无论是否有任何修改。

出于隐私的原因，有时最好混淆会话发起者的用户名和IP地址。如果隐私是个问题的话，可以选择任意的`<username>`和私有的`<unicast-address>`来填充 "o="字段，只要这些选择不影响字段的全局唯一性。

## 5.3.  会话名称("s=")

    s=<session name>

"s="字段是文本会话名称。每个会话描述必须有且仅有一个 "s=" 字段。s="字段必须不为空，并且应该包含ISO 10646字符（但也请参见 "a=charset "属性）。如果会话没有有意义的名称，则应使用 "s= " 值（即一个空格作为会话名称）。

## 5.4.  会话信息("i=")

i=<session description>

"i=" 字段提供关于会话的文字信息。每个会话描述最多只能有一个会话级的 "i="字段，每个媒体最多只能有一个 "i="字段。  如果 "a=charset "属性存在，则指定 "i="字段中使用的字符集。如果 "a=charset "属性不存在，则 "i="字段必须包含UTF-8编码的ISO 10646字符。

每个媒体定义也可以使用一个 "i="字段。在媒体定义中，"i="字段主要用于标记媒体流。它的应用场景主要是，当一个会话有同媒体类型的多个媒体流时。一个例子是两个不同的白板，一个用于幻灯片，一个用于反馈和问题。

"i="字段的目的是提供一个自由形式的，对人可读的会话描述或媒体流，但不适用于被自动数据解析。

## 5.5.  URI ("u=")

    u=<uri>

URI是WWW客户端使用的统一资源标识符[7]，URI应该是指向有关会话的附加信息的指针。这个字段是可选的，但如果它存在，它必须在第一个媒体字段之前指定。每个会话描述不允许有一个以上的URI字段。

## 5.6.  电子邮件地址和电话号码("e="和 "p=")

    e=<email-address>
    p=<phone-number>

"e=" 和 "p="两行规定了会议负责人的联系信息，不一定是创建会议公告的那个人。

加入电子邮件地址或电话号码是可选的。请注意，SDP的前一版本规定必须指定电子邮件字段或电话字段，但这一点被广泛忽视。这次修改使该规范与通常的用法相一致。

如果存在电子邮件地址或电话号码，必须在第一个媒体字段之前指定。一个会话描述可以给出一个以上的电子邮件或电话字段。

电话号码应以国际公共电信号码的形式提供(见ITU-T建议E.164)，前面加一个 "+"。如有需要，可使用空格和连字符来分割电话字段，以便于阅读。例如：

    p=+1 617 555-6011

电子邮件地址和电话号码都可以有一个可选的自由文本字符串，通常给出可能被联系的人的名字，如果有的话，但必须用括号括起来。例如:

    e=j.doe@example.com (Jane Doe)

另一种RFC 2822[29]名称引号约定也允许用于电子邮件地址和电话号码。例如:

    e=Jane Doe <j.doe@example.com>

自由文本字符串应该是ISO-10646字符集和UTF-8编码，或者是ISO-8859-1或其他编码，如果设置了适当的会话级 "a=charset "属性。

## 5.7.  连接数据("c=")

    c=<nettype> <addrtype> <connection-address>

"c="字段包含连接数据。

会话描述必须在每个媒体描述中至少包含一个 "c="字段，或者在会话级别包含一个 "c="字段，它可以包含一个单一的会话级别的 "c=" 字段和每个媒体描述的附加 "c="字段，在这种情况下，每个媒体的值覆盖了会话级别设置。

第一个子字段("`<nettype>`")是网络类型，它是一个文本字符串，表示网络的类型。最初，"IN "被定义为 "因特网 "的意思，但将来可以登记其他值（见第8节）。

第二个子字段（"`<addrtype>`"）是地址类型。这允许SDP用于不基于IP的会话。本备忘录只定义了IP4和IP6，但将来可以注册其他值（见第8节）。

第三个子字段（"`<connection-address>`"）是连接地址。根据`<addrtype>`字段的值，可以在连接地址后面添加可选的子字段。

当`<addrtype>`为IP4和IP6时，连接地址定义如下。

* 如果会话是多播的，连接地址将是一个IP多播组地址，如果会话不是多播的，则连接地址包含由附加属性字段确定的预期数据源或数据中继或数据槽的单播IP地址。虽然并不禁止在由组播公告传达的会话描述中给出单播地址，但请不要这样做。

* 使用 IPv4 组播连接地址的会话，除了组播地址外，还必须有一个生存时间（TTL）值。  TTL和地址共同定义了本次会议中发送的组播数据包的发送范围。TTL值的范围必须是0-255。虽然TTL必须是指定的，但我们不赞成用它来限定组播流量的范围；而是应用程序应该使用管理范围的地址。

会话的TTL用斜线作为分隔符附加到地址上。例如：

    c=IN IP4 224.2.36.42/127

IPv6组播不使用TTL范围，因此IPv6组播的TTL值必须不存在。IPv6的地址范围将被用来限制会议的范围。

分层或分层编码方案是一种数据流，在这种情况下，来自单一媒体源的编码被分成若干层，接收器可以选择所需的质量(以及带宽)，只需订阅这些层中的一个子集。这种分层编码通常在多个多播组中传输，以允许多播修剪。这种技术可以将不需要的流量从只需要某些层级的站点上保留下来。对于需要多个多播组的应用，我们允许使用以下符号来表示连接地址。

    <base multicast address>[/<ttl>]/<number of addresses>

如果没有给出地址数，则假定为一个。

如此分配的多播地址在基本地址之上连续分配，因此，例如：

c=IN IP4 224.2.1.1/127/3

将说明地址 224.2.1.1、224.2.1.2 和 224.2.1.3 在TTL为127时被使用。这在语义上与在媒体描述中包含多个 "c=" 行是相同的。

    c=IN IP4 224.2.1.1/127
    c=IN IP4 224.2.1.2/127
    c=IN IP4 224.2.1.3/127

类似的，一个IPv6的例子是:

    c=IN IP6 FF15::101/3

在语义上等同于:

    c=IN IP6 FF15::101
    c=IN IP6 FF15::102
    c=IN IP6 FF15::103

(记住，IPv6组播中不存在TTL字段)。

只有当多个地址或 "c=" 行在分层或分层编码方案中为不同层提供多播地址时，才可在每个介质基础上指定。它们不得指定会话级 "c=" 字段。

对于IP单播地址，不得使用上述多地址的斜线符号。

## 5.8.  带宽("b=")

    b=<bwtype>:<bandwidth>

该可选字段表示会话或媒体拟使用的带宽。`<bwtype>` 是一个字母数字修饰符，表示`<bandwidth>` 数字的含义。本规范中定义了两个值，但将来还可以注册其他值（见第 8 节和 [21]、[25]）。

* CT 如果一个会话或一个会话中的媒体的带宽与范围中隐含的带宽不同，则应为该会话提供 "b=CT:... "行，给出所使用带宽的拟议上限（"会议总 "带宽）。这样做的主要目的是提供一个近似的概念，说明两个或两个以上的会议是否可以同时存在。当在RTP中使用CT修饰符时，如果会议中有多个RTP会话，则会议总带宽指的是所有RTP会话的总带宽。

* AS 带宽被解释为特定的应用程序（它将是应用程序的最大带宽概念），通常情况下，这与应用程序的 "最大带宽 "控制中的设置一致。对于基于RTP的应用，AS给出了[19]第6.2节中定义的RTP "会话带宽"。

请注意，CT给出的是所有站点的所有媒体的总带宽数字。  AS给出的是单个站点的单个媒体的带宽数字，尽管可能有许多站点同时发送。

为`<bwtype>`名称定义了一个前缀 "X-"，这只是为了实验目的，例如：

    b=X-YZ:128

不建议使用 "X-"前缀，相反，新的修饰符应该在标准名称空间中向IANA注册。SDP解析器必须忽略带有未知修饰符的带宽字段，修饰符必须是字母数字，虽然没有长度限制，但建议它们要尽量短。

新的`<bwtype>`修饰符的定义可以指定用其他单位来解释带宽（本备忘录中定义的 "CT "和 "AS "修饰符使用默认单位）。

## 5.9.  定时("t=")

    t=<start-time> <stop-time>

"t="行指定一个会话的开始和停止时间。如果一个会话在多个不规则的时间段活动，可以使用多个 "t="行；每个额外的 "t="行指定会话活动的额外时间段。如果会话在固定的时间活动，则应在 "t="行之外和之后使用 "r="行（见下文），在这种情况下，"t="行指定重复序列的开始和停止时间。

第一个和第二个子字段分别给出会话的开始和停止时间。这些值是网络时间协议(NTP)时间值的十进制表示，单位是自1900年以来的秒数[13].要将这些值转换为UNIX时间，需要减去十进制2208988800。

NTP的时间戳在其他地方用64位的值来表示，所以在2036年的时候会出现环绕。但由于SDP使用任意长度的十进制表示法，所以这应该不会引起同样的问题（SDP时间戳必须继续计算1900年以来的秒数，NTP将使用64位限制的值）。

如果`<stop-time>`被设置为零，那么该会话就不受约束，尽管它在`<start-time>`之后才会被激活。如果`<start-time>`也是零，则会话被视为永久。

用户界面应该坚决不鼓励创建无限制和永久的会话，因为它们没有提供关于会话何时真正终止的信息，因此使调度变得困难。

当向用户显示未超时的无时间范围会话时，一般假设无时间范围的会话只在当前时间或会话开始时间后半小时内有效，以较晚者为准。如果需要比这更多的行为，应该给出一个结束时间，并在新的信息变得可用时适当地修改会话真正应该结束的时间。

## 5.10.  重复时间("r=")

    r=<repeat interval> <active duration> <offsets from start-time>

"r="字段指定会话的重复时间。例如，如果一个会话在周一上午10点和周二上午11点活动，每周活动一小时，持续三个月，那么对应的 "t="字段中的`<start-time>`将是第一个周一上午10点的NTP表示，`<repeat interval>`将是1周，`<active duration>`将是1小时，偏移量将是0和25小时。  相应的 "t="字段的停止时间将是三个月后最后一次会话结束的NTP时间表示。默认情况下，所有字段的单位都是秒，所以 "r="和 "t="字段的单位可能是如下：

    t=3034423619 3042462419
    r=604800 3600 0 90000

为了使描述更紧凑，时间也可以用天、小时或分钟的单位给出。这些单位的语法是一个数字，后面紧跟着一个大小写敏感字符，不允许使用小数单位，而应使用较小的单位。允许使用以下单位规格字符：

    d - days (86400 seconds)
    h - hours (3600 seconds)
    m - minutes (60 seconds)
    s - seconds (allowed for completeness)

因此，上述会话公告也可以写成。

    r=7d 1h 0 25h

月度和年度重复不能直接用单个SDP重复时间来指定，而应使用单独的 "t="字段来明确列出会话时间。

5.11.  时区("z=")

    z=<adjustment time> <offset> <adjustment time> <offset> ....

要安排一个跨越从夏令时到标准时或反之的重复会话，必须指定基准时间的偏移量。之所以需要这样做，是因为不同的时区会在一天中的不同时间改变时间，不同的国家会在不同的日期改变夏令时的时间，而有些国家根本就没有夏令时。

因此，为了安排一个在同一时间的冬季和夏季的会话，必须能够毫不含糊地指定由谁的时区来安排一个会话。为了简化接收者的这项任务，我们允许发送者指定时区调整发生的NTP时间，以及与会话首次安排时的偏移量。z="字段允许发送者指定这些调整时间和与基准时间的偏移量的列表。

以下是一个例子：

    z=2882844526 -1h 2898848070 0

这表示在时间2882844526时，计算会话重复时间的时基向后移动1小时，在时间2898848070时，恢复会话的原始时基。调整始终是相对于指定的开始时间而言的 -- 它们不是累积的。  调整适用于会话描述中的所有 "t=" 和 "r=" 行。

如果一届会议可能持续数年，预计会定期修改会议公告，而不是在一次会议公告中传送数年的调整。

## 5.12.  加密密钥("k=")

    k=<method>
    k=<method>:<encryption key>

如果通过安全和可信的信道传输，可以使用会话描述协议来传递加密密钥。密钥字段("k=")提供了一个简单的密钥交换机制，尽管这主要是为了与旧的实例兼容而支持的，并且不推荐使用它。目前正在进行定义新的密钥交换机制的工作，以便与SDP一起使用[27][28]，预计新的应用将使用这些机制。

在第一个媒体条目之前（在这种情况下，它适用于会话中的所有媒体），或根据需要为每个媒体条目设置密钥字段。密钥的格式及其用法不在本文档的范围内，而且密钥字段没有提供任何方式来指示要使用的加密算法、密钥类型或其他有关密钥的信息：这些信息被认为是由使用SDP的上层协议提供的。如果需要在SDP中传达这些信息，就应该使用前面提到的扩展。许多安全协议需要两个密钥：一个用于保密性，另一个用于完整性。本规范不支持两个密钥的传输。

该方法表示通过外部手段或从给定的加密密钥中获取可用密钥的机制，定义了以下方法：

* `k=clear:<encryption key>`

    加密密钥包含在这个密钥字段中，未作任何转换，除非能保证SDP是通过安全通道传输的，否则绝不能使用这种方法。加密密钥根据字符集属性被解释为文本；使用 "k=base64:" 方法来传递SDP中禁止的其他字符。

* `k=base64:<encoded encryption key>`

    加密密钥包含在这个密钥字段中，但已经被base64编码[12]，因为它包含了SDP中禁止的字符。  除非能保证SDP是通过安全通道传输的，否则一定不要使用这种方法。

* `k=uri:<URI to obtain key>`

    密钥字段中包含一个统一资源标识符，URI指的是包含密钥的数据，在返回密钥之前可能需要额外的认证。当向给定的URI提出请求时，回复应指定密钥的编码。URI通常是受安全套接字层传输层安全(SSL/TLS)保护的HTTP URI("https:")，尽管这不是必需的。

* `k=prompt` 

    这个SDP描述中不包含密钥，但这个密钥字段所引用的会话或媒体流是加密的。当用户试图加入会话时，应提示用户输入密钥，然后使用这个用户提供的密钥对媒体流进行加密。不推荐使用用户指定的密钥，因为这种密钥往往具有较弱的安全属性。

除非能保证SDP是通过一个安全可信的信道传输的，否则一定不要使用密钥字段。这种信道的一个例子是嵌入在S/MIME消息或受TLS保护的HTTP会话中的SDP。重要的是要确保安全通道是与被授权加入会话的一方，而不是中间人：如果使用了一个缓存代理服务器，重要的是要确保代理是可信的或无法访问SDP。

## 5.13.  属性("a=")

    a=<attribute>
    a=<attribute>:<value>

属性是扩展SDP的主要手段。属性可能被定义为 "会话级 "属性、"媒体级 "属性，或者两者兼而有之。

一个媒体描述可以有任何数量的媒体特定属性（"a="字段）。这些属性被称为 "媒体级 "属性，增加了有关媒体流的信息。还可以在第一个媒体字段之前添加属性字段；这些 "会话级 "属性传递适用于整个会议而不是单个媒体的附加信息。

属性字段可以有两种形式。

* 属性性质的简单形式是 "`a=<flag>`"，这些都是二值属性，属性的存在表明该属性是会话的一个性质。一个例子可能是 "a=recvonly"。

* 值属性的形式是 "`a=<attribute>:<value>`"。  例如，一块白板的值属性可以是 "a=orient:landscape"，属性的解释取决于被调用的媒体工具，因此会话描述的接收者应该可以配置其对会话描述的解释，特别是对属性的解释。

属性名必须使用ISO-10646UTF-8的US-ASCII子集。

属性值是8-bit字符串，可以使用任何8-bit的值，但0x00 (Nul)、0x0A (LF)和0x0D (CR)除外。默认情况下，属性值将被解释为ISO-10646字符集和UTF-8编码。与其他文本字段不同，属性值通常不受 "charset "属性的影响，因为这将使与已知值的比较变得困难。然而，当一个属性被定义时，它可以被定义为依赖于字符集，在这种情况下，它的值应该用会话字符集而不是ISO-10646来解释。

属性必须在IANA注册（见第8节）。  如果收到的属性不被理解，接收者必须忽略它。

## 5.14.  媒体描述("m=")

    m=<media> <port> <proto> <fmt>......。

一个会话描述可以包含多个媒体描述，每个媒体描述以 "m="字段开始，并以下一个 "m="字段或会话描述的结尾结束，一个媒体字段有几个子字段。

`<media>` 是媒体类型。  目前定义的媒体是 "audio","video", "text", "application", 和 "message"，尽管这个列表将来可能会扩展（见第8节）。

`<port>` 是发送媒体流的传输端口。传输端口的含义取决于相关的 "c=" 字段中所使用的网络，以及媒体字段的 `<proto>` 子字段中定义的传输协议。可以通过算法从基础媒体端口导出，也可以在一个单独的属性中指定(例如，"a=rtcp:"，如[22]中定义的)。

如果使用了非相邻的端口，或者它们没有遵循偶数RTP端口和奇数RTCP端口的对等规则，则必须使用 "a=rtcp:"属性。应用程序如果被要求将媒体发送到奇数的<port>，并且存在 "a=rtcp:"，则必须不从RTP端口中减去1：也就是说，它们必须将RTP发送到<port>中指示的端口，并将RTCP发送到 "a=rtcp"属性中指示的端口。

对于将分级编码的流发送到单播地址的应用，可能需要指定多个传输端口。这一点与 "c="字段中的IP多播地址使用的符号类似。

    m=<media> <port>/<number of ports> <proto> <fmt> ...。

在这种情况下，使用的端口取决于传输协议。对于RTP，默认是只有偶数端口用于数据，相应的加1后的奇数端口用于属于RTP会话的RTCP，`<number of ports>` 表示RTP会话的数量。例如：

    m=video 49170/2 RTP/AVP 31

将指定端口49170和49171组成一个RTP/RTCP对，49172和49173组成第二个RTP/RTCP对，同时RTP/AVP是传输协议，31是格式（见下文）。如果需要非相邻的端口，则必须使用一个单独的属性(例如，"a=rtcp:"，如[22]中定义的)来表示。

如果在 "c="字段中指定了多个地址，并且在 "m="字段中指定了多个端口，则意味着从端口到相应地址的一对一映射。例如

    c=IN IP4 224.2.1.1/127/2
    m=video 49170/2 RTP/AVP 31

将意味着地址224.2.1.1与端口49170和49171一起使用，地址224.2.1.2与端口49172和49173一起使用。

使用同一传输地址的多个 "m="行的语义是未定义的。这意味着没有通过这种方式定义的隐式分组，而应该使用一个明确的分组框架(例如[18])来表达预期的语义。

`<proto>` 是传输协议。传输协议的含义取决于相关 "c="字段中的地址类型。因此，IP4 的 "c="字段表示传输协议在 IP4 上运行。以下是已定义的传输协议，但可通过向 IANA 注册新协议来扩展（见第 8 节）。

* udp：表示通过UDP运行的未指定协议。

* RTP/AVP：表示在UDP上运行的最小控制的音频和视频会议RTP配置文件[19]下使用的RTP。

* RTP/SAVP：表示在UDP上运行的安全实时传输协议[23]。

除了媒体格式之外，指定传输协议的主要原因是，即使网络协议相同，相同的标准媒体格式也可以通过不同的传输协议进行传输 -- 历史上的一个例子是 Pulse Code Modulation (PCM)音频和RTP PCM音频；另一个例子可能是TCP/RTP PCM音频。此外，还可以采用传输协议专用但格式无关的中继和监控工具。

`<fmt>` 是媒体格式描述。第四个子字段和任何子字段描述媒体的格式。媒体格式的解释取决于 `<proto>` 子字段的值。

如果`<proto>` 子字段是 "RTP/AVP "或 "RTP/SAVP"，则`<fmt>`子字段包含RTP有效载荷类型号。当给定payload类型编号列表时，这意味着所有这些payload格式都可以在会话中使用，但这些格式中的第一种应该作为会话的默认格式。对于动态有效载荷类型， "a=rtpmap:"属性（见第6节）应该被用于从RTP有效载荷类型编号映射到标识有效载荷格式的媒体编码名称。"a=fmtp:"属性可以用来指定格式参数（见第6节）。

如果 `<proto>` 子字段为 "udp"，则 `<fmt>` 子字段必须引用描述 "audio","video", "text", "application", 或 "message" [顶级介质类型下的格式的介质类型](https://www.iana.org/assignments/media-types/media-types.xhtml#video)。媒体类型注册必须定义用于UDP传输的数据包格式。

对于使用其他传输协议的媒体，`<fmt>` 字段是特定于协议的。注册新协议时，必须定义`<fmt>`子字段的解释规则（见第8.2.2节）。

# 6.SDP属性

协议定义了以下属性。由于应用程序编写者可以根据需要添加新的属性，所以这个列表并不详尽。新属性的注册程序在第8.2.4节中定义。

* `a=cat:<category>`

    这个属性给出了会话的采用点分隔的层次类别。这是为了让接收者能够通过类别来过滤不想要的会话。没有一个集中的类别注册表。它是一个会话级别的属性，并且它不依赖于字符集。

* `a=keywds:<keywords>`

    像cat属性一样，这是为了帮助接收方识别想要的会话。这允许接收者根据描述会话目的的关键字来选择有趣的会话；没有关键字的中央注册表。它是一个会话级别的属性。它是一个依赖于字符集的属性，这意味着如果指定了会话描述，它的值应该用指定的字符集来解释，或者默认为ISO 10646UTF-8。

* `a=tool:<name and version of tool>`

    这给出了用于创建会话描述的工具的名称和版本号。这是一个会话级别的属性，它不依赖于字符集。

* `a=ptime:<packet time>`

    这给出了一个数据包中介质所代表的时间长度，单位为毫秒。这可能只对音频数据有意义，但也可以用于其他媒体类型。对RTP或vat音频进行解码时，应该不需要知道ptime，它是对音频编码包化的一个建议。它是一个媒体级别的属性，它不依赖于字符集。

* `a=maxptime:<maximum packet time>`

    这给出了每个数据包可以封装的最大介质量，以毫秒为单位的时间。该时间必须以数据包中的媒体所代表的时间之和来计算。对于基于帧的编解码器，时间应该是帧大小的整数倍。这个属性可能只对音频数据有意义，但也可以用于其他媒体类型。  这是一个媒体级别的属性，它不依赖于字符集。请注意，这个属性是在RFC 2327之后引入的，如果没有更新实现的话，将忽略这个属性。

* `a=rtpmap:<payload type> <encoding name>/<clock rate> [/<encoding parameters>]`

    这个属性从一个RTP有效载荷类型号（如在 "m="行中使用）映射到一个表示要使用的有效载荷格式的编码名称。它还提供了时钟速率和编码参数的信息。它是一个媒体级属性，不依赖于字符集。

    虽然 RTP profile 可以对有效载荷格式的有效载荷类型号进行静态分配，但更常见的是使用 "a=rtpmap:" 属性进行动态分配。作为静态有效载荷类型的一个例子，考虑 u-law PCM 编码的8kHz采样单通道音频，这在RTP Audio/Video profile 中被完全定义，有效载荷类型号为 0，所以不需要 "a=rtpmap: "属性，所以发送到UDP端口49232的这种流的媒体可以被指定为:

        m=audio 49232 RTP/AVP 0
    
    动态有效载荷类型的例子是16位线性编码立体声音频，采样频率为16 kHz。如果我们希望对该流使用动态RTP/AVP有效载荷类型98，则需要额外的信息对其进行解码。

        m=audio 49232 RTP/AVP 98
        a=rtpmap:98 L16/16000/2
        
    每个媒体格式可以定义一个rtpmap属性。  因此，我们可能会有以下内容:

      m=audio 49230 RTP/AVP 96 97 98
      a=rtpmap:96 L8/8000
      a=rtpmap:97 L16/8000
      a=rtpmap:98 L16/11025/2

    RTP profile 如果指定使用动态有效载荷类型，必须定义有效编码名称的集合，如果该profile 要与 SDP 一起使用，则必须定义注册编码名称的方法。RTP/AVP 和 RTP/SAVP profile 在 "m=" 行中表示的顶级媒体类型下，使用媒体子类型作为编码名称。  在上面的例子中，媒体类型是 "audio/l8 "和 "audio/l16"。

    对于音频流，`<encoding parameters>`表示音频通道数。如果通道数为1，则该参数可以省略，不需要额外参数。

    对于视频流，目前没有指定编码参数。

    未来可以定义额外的编码参数，但不应该添加编解码器专用参数。添加到 "a=rtpmap: "属性中的参数应该是会话目录所需的参数，以便选择合适的介质参与会话。特定于编解码器的参数应该添加到其他属性中（例如，"a=fmtp:"）。

    注意：RTP音频格式通常不包括每个数据包的采样数信息。如果需要使用一个非默认的（如RTP Audio Video Profile中定义的）数据包，则使用 "ptime "属性，如上所述。

* a=recvonly

    这指定工具应在适用的情况下以只接收模式启动。它可以是会话或媒体级别的属性，并且不依赖于字符集。注意recvonly只适用于媒体，而不是任何相关的控制协议（例如，一个基于RTP的系统在recvonly模式下应该仍然发送RTCP数据包）。

* a=sendrecv

    这指定工具应在发送和接收模式下启动。这对于交互式会议来说是必要的，因为这些工具默认为只接收模式。它可以是会话或媒体级别的属性，而且它不依赖于charset。

    如果 "sendonly"、"recvonly"、"inactive "和 "sendrecv "这几个属性都不存在，那么 "sendrecv "应该被认为是非"broadcast "或 "H332 "类型会议的默认属性（见下文）。

* a=sendonly

    这指定了工具应该在只发送模式下启动。  一个例子是，当一个流量目标与一个流量源使用不同的单播地址时，可能会使用两个媒体描述，一个是sendonly，一个是recvonly。它可以是会话级或媒体级属性，但通常只作为媒体属性使用。它不依赖于charset。请注意，sendonly只适用于媒体，任何相关的控制协议(例如RTCP)仍然应该像正常的那样被接收和处理。

* a=inactive

    这说明工具应该在非活动模式下启动。这对于交互式会议来说是必要的，因为在交互式会议中，用户可以让其他用户处于等待状态。不通过非活动媒体流发送任何媒体。请注意，一个基于RTP的系统即使在非活动状态下启动，也应该发送RTCP。它可以是会话或媒体级别的属性，而且它不依赖于charset。

* `a=orient:<orientation>` 

    通常这只用于白板或演示工具。  它指定了屏幕上工作区的方向。它是一个媒体级属性。允许的值是 "纵向"、"横向 "和 "seascape"（上下翻转）。它不依赖于字符集。

* `a=type:<conference type>`

    这指定了会议的类型，建议使用 "broadcast", "meeting", "moderated", "test", 和 "H332"。  建议值为 "广播"、"会议"、"主持"、"测试 "和 "H332"。"recvonly "应该是 "type:broadcast "会议的默认值，"type:meeting "应该意味着 "sendrecv"，"type:modified "应该表示使用 `floor control tool`，并且媒体工具被启动，以便使新加入会议的站点静音。

    指定属性 "type:H332" 表示这个松散耦合的会话是ITU H.332规范[26]中定义的H.332会话的一部分。媒体工应该以 "recvonly "启动。

    指定属性 "type:test" 是作为一种暗示，除非明确要求，否则接收者可以安全地不向用户显示这个会话描述。

    类型属性是一个会话级别的属性，它不依赖于charset。

* `a=charset:<character set>`

    指定用于显示会话名称和信息数据的字符集。默认情况下，使用UTF-8编码的ISO-10646字符集。例如，ISO 8859-1可以通过以下SDP属性来指定。

        a=charset:ISO-8859-1
        
    这是一个会话级属性，不依赖于charset。指定的字符集必须是在IANA注册的字符集之一，如ISO-8859-1。字符集标识符是一个 US-ASCII 字符串，必须使用不区分大小写的比较方法与 IANA 的标识符进行比较。如果该标识符不被识别或不被支持，所有受其影响的字符串都应被视为8-bit字符串。

    请注意，指定的字符集必须仍然禁止使用 0x00 (Nul)、0x0A (LF) 和 0x0d (CR) 字节。需要使用这些字符的字符集必须定义一个引号机制，以防止这些字节出现在文本字段中。

* `a=sdplang:<language tag>`

    这可以是一个会话级属性或媒体级属性。作为会话级属性，它指定会话描述的语言。作为媒体级属性，它指定了与该媒体相关联的任何媒体级SDP信息字段的语言。如果会话描述或媒体中使用多种语言，则可以在会话或媒体级别提供多个 sdp 语言属性，在这种情况下，属性的顺序表示会话或媒体中各种语言的重要性顺序，从最重要到最不重要。

    一般来说，不鼓励发送由多种语言组成的会话描述，相反，应该发送多个描述来描述会话，每个语言一个。然而，这并不是所有的传输机制都能做到的，所以允许发送多个 sdplang 属性，尽管不推荐。

    sdplang "属性值必须是一个US-ASCII[9]的RFC 3066语言标签。它不依赖于charset属性。当一个会话的范围足够大，可以跨越地理边界，无法假定接收者的语言，或者会话的语言与当地假定的标准不同时，应该指定 "sdplang "属性。

* `a=lang:<language tag>`

    这可以是一个会话级属性或媒体级属性。作为会话级属性，它指定了被描述的会话的默认语言。作为媒体级属性，它指定了该媒体的语言，覆盖了任何指定的会话级语言。如果会话描述或媒体使用多种语言，可以在会话或媒体级别提供多种语言属性，在这种情况下，属性的顺序表示会话或媒体中各种语言的重要性顺序，从最重要到最不重要。

    "lang "属性值必须是一个单一的RFC 3066语言标记，用US-ASCII[9]表示。  它不依赖于charset属性。当session的范围足以跨越地理边界，无法假定接收者的语言，或者session的语言与当地假定的语言不同时，应该指定 "lang "属性。

* `a=framerate:<frame rate>`

    这给出了以frames/sec为单位的最大视频帧率。允许使用"<整数>.<分数>"来表示分数值的小数，它是媒体级属性，只为视频媒体定义，不依赖于字符集。

* `a=quality:<quality>`

    这给出了编码质量的建议，是一个整数值。对于视频来说，质量属性的目的是指定帧速率和静态图像质量之间的非默认权衡。对于视频来说，该值的范围是0到10，其建议含义如下。

        10 - 压缩方案所能提供的最佳静止图像质量.
        5 - 没有质量建议时的默认行为.
        0 - 编解码器设计者认为仍然可用的最差静止图像质量。

    这是一个媒体级别的属性，它不依赖于charset。

* `a=fmtp:<format> <format specific parameters>`

    该属性允许以SDP不必理解的方式传递特定格式的参数。该格式必须是为媒体指定的格式之一。特定格式的参数可以是任何一组需要由 SDP 传递的参数，并将其不变地提供给将使用此格式的媒体工具。每种格式最多允许有一个该属性的实例。

    这是一个媒体级别的属性，它不依赖于charset。

# 7. 安全因素考虑

SDP经常与 Session Initiation Protocol[15] 一起使用 "offer/answer"模型[17]来商定单播会话的参数。当以这种方式使用时，需要有这样的一些安全考虑。

SDP是一种描述多媒体会话的会话描述格式。接收SDP电文并对其采取行动的实体应意识到，除非会话描述是由经过认证的传输协议从已知和受信任的来源获得的，否则它是不可信任的。  许多不同的传输协议可用于分发会话描述，不同传输协议的认证性质也不同。对于一些传输协议，往往没有部署安全功能。如果会话描述不是以可信的方式获得的，终端应该小心，因为如果存在攻击，则接收到的媒体会话可能不是预期的，媒体被发送到的目的地可能不是预期的，会话的任何参数可能是不正确的，或者媒体的安全性可能会受到损害。终端应该在考虑到应用的安全风险和用户偏好的情况下做出明智的决定，并可能决定询问用户是否接受会话。

可用于分发会话描述的一种传输方式是会话公告协议（SAP）。SAP提供了加密和认证机制，但由于会话公告的性质，很可能在许多情况下，会话公告的发起人无法得到认证，因为公告的接收者以前不知道发起人，也因为没有通用的公钥基础设施。

当通过未经认证的传输机制或从不信任的一方收到会话描述时，解析会话的软件应采取一些预防措施。会话描述包含了启动接收器系统上的软件所需的信息，解析会话描述的软件必须不能启动其他软件，除非该软件被特别配置为适合参与多媒体会话的软件。在未告知用户将启动这些软件并得到用户同意的情况下，解析会话描述的软件在用户系统上启动适合参与多媒体会话的软件的行为，通常被认为是违法的。因此，通过会话公告、电子邮件、会话邀请或WWW网页到达的会话描述不得将用户带入交互式多媒体会话，除非用户已经明确地预先授权了这种行为。由于并不总是能够简单地判断一个会话是否是交互式的，所以不确定的应用程序应该假设会话是交互式的。

在本规范中，没有任何属性可以让会话描述的接受者被告知以默认为传输的模式启动多媒体工具。在某些情况下，定义这样的属性可能是合适的。如果这样做了，应用程序在解析包含这些属性的会话描述时，应该忽略这些属性，或者告知用户加入这个会话将导致多媒体数据的自动传输。对于未知属性的默认行为是忽略它。

在某些环境中，中间系统拦截和分析其他信令协议中包含的会话描述已变得很普遍。这样做的目的很多，包括但不限于打开防火墙的洞，以允许媒体流通过，或有选择地标记、优先处理或阻止流量。在某些情况下，这种中间系统可能会修改会话描述，例如，使会话描述的内容与动态创建的NAT绑定相匹配。这些行为并不推荐，除非会话描述的传达方式允许中间系统进行适当的检查，以建立会话描述的真实性，以及其源头建立这种通信会话的权威性。SDP本身并不包括足够的信息来实现这些检查：它们取决于封装协议（如SIP或RTSP）。

使用 "k="字段会带来很大的安全风险，因为它传递的是透明的会话加密密钥。除非能保证传递SDP的信道既是私有的又是经过认证的，否则绝不能用SDP来传递密钥材料。此外，"k="行没有提供任何方式来指示或协商加密密钥算法。由于它只提供了一个单对称密钥，而不是用于保密性和完整性的独立密钥，因此它的作用受到严重限制。如第5.12节所述，不建议使用 "k="行。

# 8.  IANA的考虑

## 8.1.  "application/sdp "媒体类型

要更新RFC 2327 中注册的一个媒体类型，定义如下。

      To: ietf-types@iana.org
      Subject: Registration of media type "application/sdp"

      Type name: application

      Subtype name: sdp

      Required parameters: None.

      Optional parameters: None.

      Encoding considerations:
         SDP files are primarily UTF-8 format text.  The "a=charset:"
         attribute may be used to signal the presence of other
         character sets in certain parts of an SDP file (see
         Section 6 of RFC 4566).  Arbitrary binary content cannot
         be directly represented in SDP.

      Security considerations:
         See Section 7 of RFC 4566

      Interoperability considerations:
         See RFC 4566

      Published specification:
         See RFC 4566

      Applications which use this media type:
         Voice over IP, video teleconferencing, streaming media, instant
         messaging, among others.  See also Section 3 of RFC 4566.

      Additional information:

      Magic number(s):   None.
      File extension(s): The extension ".sdp" is commonly used.
      Macintosh File Type Code(s): "sdp "

      Person & email address to contact for further information:
         Mark Handley  <M.Handley@cs.ucl.ac.uk>
         Colin Perkins <csp@csperkins.org>
         IETF MMUSIC working group <mmusic@ietf.org>

      Intended usage: COMMON

      Author/Change controller:
         Authors of RFC 4566
         IETF MMUSIC working group delegated from the IESG

## 8.2.  注册参数

有七个字段名可以在IANA注册。使用SDP规范Backus-Naur Form(BNF)中的术语，它们是 "media"、"proto"、"fmt"、"att-field"、"bwtype"、"nettype "和 "addrtype"。

### 8.2.1. 媒体类型("media")

这套媒体类型的数量不多，除非在极少数情况下，否则不应扩大。媒体名称的规则与顶层媒体内容类型的规则相同，在可能的情况下，SDP的名称应与MIME的名称相同。除了现有的顶层媒体内容类型以外的媒体，必须为新的顶层内容类型注册一个标准跟踪RFC，而且注册时必须提供良好的理由，说明为什么现有的媒体名称不合适(RFC 2434[8]的 "标准行动 "政策)。

该备忘录登记了 "audio", "video", "text","application", 和 "message" 等媒体类型。

注：在本规范的前一版本[6]中，"control "和 "data "被列为有效的媒体类型，但是，它们的符号从未被完全指定，而且它们没有被广泛使用。如果这些媒体类型在将来被认为是有用的，那么必须制定一个标准跟踪RFC来记录它们的使用。在这之前，应用程序不应该使用这些类型，也不应该在 SIP 能力声明中声明对它们的支持（尽管它们存在于RFC 3840创建的注册表中）。

### 8.2.2.  传输协议("proto") 

"proto "字段描述使用的传输协议。该字段应参考一个标准跟踪协议RFC。本备忘录注册三个值。 "RTP/AVP "是对在UDP/IP上运行的RTP Profile for Audio and Video Conferences with Minimal Control[20]下使用的RTP[19]的引用，"RTP/SAVP "是对Secure Real-time Transport Protocol[23]的引用，"udp "表示UDP上的一个未指明的协议。

如果将来定义了其他的RTP配置文件，它们的 "proto "名称应该以同样的方式指定。  例如，一个简称为 "XYZ "的RTP配置文件将由 "RTP/XYZ "的 "proto "字段来表示。

新的传输协议应向IANA注册。注册时必须提及描述协议的RFC。这种RFC可以是实验性的，也可以是信息性的，但最好是标准跟踪的。注册还必须定义管理其 "fmt "命名空间的规则（见下文）。

### 8.2.3.  媒体格式("fmt")

由 "proto "字段定义的每个传输协议都有一个相关的 "fmt "命名空间，描述该协议可能传递的媒体格式。格式涵盖了多媒体会话中可能要传输的所有编码。

"RTP/AVP "和 "RTP/SAVP " profile 下的RTP有效载荷格式必须使用有效载荷类型号作为其 "fmt "值。如果有效载荷类型号是由该会话描述动态分配的，则必须包含一个附加的 "rtpmap "属性，以指定格式名和参数，这些格式和参数由payload格式的媒体类型注册所定义。建议将其他 RTP 配置文件注册为 SDP 传输协议（与 RTP 结合），为 "fmt "命名空间指定相同的规则。

对于 "udp "协议，应注册新的格式。鼓励对格式使用现有的媒体子类型。如果没有媒体子类型，建议通过IETF程序[31]注册一个合适的媒体子类型，制作或引用定义该格式传输协议的标准跟踪RFC。

对于其他协议，可以根据相关 "proto "规范的规则注册格式。

新格式的注册必须说明它们适用于哪些传输协议。

### 8.2.4.  属性名称("att-field")

属性字段名称("att-field")必须在IANA注册并记录在案，因为同一名称下的冲突属性会引起明显的问题。SDP中未知的属性被简单地忽略，但冲突的属性使协议变得支离破碎是个严重的问题。

根据RFC 2434的 "规范要求"，新的属性注册被接受，条件是该规范包括以下信息。

* 联系人姓名、电子邮件地址和电话号码
* 属性名称（因为它将出现在SDP中）
* 英文长式属性名称
* 属性类型（会话级别、媒体级别或两者）
* 属性值是否受制于charset属性
* 一段关于属性目的的解释
* 此属性的适当属性值的规范

以上是IANA所接受的最低限度。预期会被广泛使用和互操作性的属性，应该用标准跟踪的RFC来记录，它能更精确地指定属性。

注册的提交者应确保该规范符合SDP属性的精神，最值得注意的是，该属性是独立于平台的，即它不对操作系统作出隐含的假设，也不以可能妨碍互操作性的方式命名特定的软件。

IANA已经注册了以下初始属性名("att-field "值)，其定义见本备忘录第6节(这些定义更新了RFC 2327中的定义)。

      Name      | Session or Media level? | Dependent on charset?
      ----------+-------------------------+----------------------
      cat       | Session                 | No
      keywds    | Session                 | Yes
      tool      | Session                 | No
      ptime     | Media                   | No
      maxptime  | Media                   | No
      rtpmap    | Media                   | No
      recvonly  | Either                  | No
      sendrecv  | Either                  | No
      sendonly  | Either                  | No
      inactive  | Either                  | No
      orient    | Media                   | No
      type      | Session                 | No
      charset   | Session                 | No
      sdplang   | Either                  | No
      lang      | Either                  | No
      framerate | Media                   | No
      quality   | Media                   | No
      fmtp      | Media                   | No

### 8.2.5. 带宽规格（"bwtype"）

强烈反对带宽规格的滥用。

新的带宽规格（"bwtype "字段）必须向IANA注册。提交的文件必须引用标准跟踪RFC，准确地说明带宽指定器的语义，并说明何时应使用，以及为什么现有注册的带宽指定器不够用。

IANA已经按照本备忘录第5.8节的定义注册了 "CT "和 "AS "这两个带宽指标（这些定义更新了RFC 2327中的定义）。

### 8.2.6.  新的网络类型（"nettype"）

如果SDP需要在非互联网环境中使用，可以向IANA注册新的网络类型（"nettype "字段），虽然这些类型通常不是IANA的专利，但在某些情况下，互联网应用需要与非互联网应用进行互操作，例如将互联网电话呼叫接入公共交换电话网（PSTN）。在注册一个新的网络类型之前，至少要注册一个与该网络类型一起使用的地址类型。新的网络类型注册必须参考一个RFC，该RFC给出了网络类型和地址类型的细节，并规定了它们的使用方式和时间。

IANA已经注册了代表互联网的网络类型 "IN"，其定义见本备忘录第5.2和5.7节（这些定义更新了RFC 2327中的定义）。

### 8.2.7.  新的地址类型("addrtype")

可以向IANA注册。地址类型只有在网络类型的情况下才有意义，任何地址类型的注册必须指定一个已注册的网络类型，或与网络类型注册一起提交。新的地址类型注册必须参考RFC，它给出了地址类型的详细语法。预计地址类型不会频繁注册。

IANA已经注册了地址类型 "IP4 "和 "IP6"，其定义见本备忘录第5.2和5.7节（这些定义更新了RFC 2327中的定义）。

### 8.2.8.  注册程序

在注册SDP "media"、"proto"、"fmt"、"bwtype"、"nettype" 和 "addrtype" 字段的RFC文档中，作者必须包含以下信息，以便IANA将其放入适当的注册表中。

* 联系人姓名、电子邮件地址和电话号码
* 被注册的名称（按其在 SDP 中的显示方式）
* 英文长式名称
* 名称类型（"media"、"proto"、"fmt"、"bwtype"、"nettype "或 "addrtype"）
* 一段关于注册名称目的的解释
* 对注册名称规范的引用（这通常是 RFC 编号）

IANA 可将任何注册提交 IESG 审查，并可要求在注册前进行修改。

## 8.3.  加密密钥访问方法

IANA以前有一个SDP加密密钥访问方法（"enckey"）名称表。这个表已经过时了，因为 "k="一行是不可扩展的。新的注册必须不被接受。

# 9. SDP语法

本节为SDP提供了一个增强的BNF语法。ABNF在[4]中定义。

    ; SDP Syntax
    session-description = proto-version
                          origin-field
                          session-name-field
                          information-field
                          uri-field
                          email-fields
                          phone-fields
                          connection-field
                          bandwidth-fields
                          time-fields
                          key-field
                          attribute-fields
                          media-descriptions
  
    proto-version =       %x76 "=" 1*DIGIT CRLF
                          ;this memo describes version 0
  
    origin-field =        %x6f "=" username SP sess-id SP sess-version SP
                          nettype SP addrtype SP unicast-address CRLF
  
    session-name-field =  %x73 "=" text CRLF
  
    information-field =   [%x69 "=" text CRLF]
  
    uri-field =           [%x75 "=" uri CRLF]
  
    email-fields =        *(%x65 "=" email-address CRLF)
  
    phone-fields =        *(%x70 "=" phone-number CRLF)
  
    connection-field =    [%x63 "=" nettype SP addrtype SP
                          connection-address CRLF]
                          ;a connection field must be present
                          ;in every media description or at the
                          ;session-level
    bandwidth-fields =    *(%x62 "=" bwtype ":" bandwidth CRLF)
  
    time-fields =         1*( %x74 "=" start-time SP stop-time
                          *(CRLF repeat-fields) CRLF)
                          [zone-adjustments CRLF]
  
    repeat-fields =       %x72 "=" repeat-interval SP typed-time
                          1*(SP typed-time)
  
    zone-adjustments =    %x7a "=" time SP ["-"] typed-time
                          *(SP time SP ["-"] typed-time)
  
    key-field =           [%x6b "=" key-type CRLF]
  
    attribute-fields =    *(%x61 "=" attribute CRLF)
  
    media-descriptions =  *( media-field
                          information-field
                          *connection-field
                          bandwidth-fields
                          key-field
                          attribute-fields )
  
    media-field =         %x6d "=" media SP port ["/" integer]
                          SP proto 1*(SP fmt) CRLF
  
    ; sub-rules of 'o='
    username =            non-ws-string
                          ;pretty wide definition, but doesn't
                          ;include space
  
    sess-id =             1*DIGIT
                          ;should be unique for this username/host
  
    sess-version =        1*DIGIT
  
    nettype =             token
                          ;typically "IN"
  
    addrtype =            token
                          ;typically "IP4" or "IP6"
  
    ; sub-rules of 'u='
    uri =                 URI-reference
                          ; see RFC 3986
    ; sub-rules of 'e=', see RFC 2822 for definitions
    email-address        = address-and-comment / dispname-and-address
                           / addr-spec
    address-and-comment  = addr-spec 1*SP "(" 1*email-safe ")"
    dispname-and-address = 1*email-safe 1*SP "<" addr-spec ">"
  
    ; sub-rules of 'p='
    phone-number =        phone *SP "(" 1*email-safe ")" /
                          1*email-safe "<" phone ">" /
                          phone
  
    phone =               ["+"] DIGIT 1*(SP / "-" / DIGIT)
  
    ; sub-rules of 'c='
    connection-address =  multicast-address / unicast-address
  
    ; sub-rules of 'b='
    bwtype =              token
  
    bandwidth =           1*DIGIT
  
    ; sub-rules of 't='
    start-time =          time / "0"
  
    stop-time =           time / "0"
  
    time =                POS-DIGIT 9*DIGIT
                          ; Decimal representation of NTP time in
                          ; seconds since 1900.  The representation
                          ; of NTP time is an unbounded length field
                          ; containing at least 10 digits.  Unlike the
                          ; 64-bit representation used elsewhere, time
                          ; in SDP does not wrap in the year 2036.
  
    ; sub-rules of 'r=' and 'z='
    repeat-interval =     POS-DIGIT *DIGIT [fixed-len-time-unit]
  
    typed-time =          1*DIGIT [fixed-len-time-unit]
  
    fixed-len-time-unit = %x64 / %x68 / %x6d / %x73
  
    ; sub-rules of 'k='
    key-type =            %x70 %x72 %x6f %x6d %x70 %x74 /     ; "prompt"
                          %x63 %x6c %x65 %x61 %x72 ":" text / ; "clear:"
                          %x62 %x61 %x73 %x65 "64:" base64 /  ; "base64:"
                          %x75 %x72 %x69 ":" uri              ; "uri:"
  
    base64      =         *base64-unit [base64-pad]
    base64-unit =         4base64-char
    base64-pad  =         2base64-char "==" / 3base64-char "="
    base64-char =         ALPHA / DIGIT / "+" / "/"
  
    ; sub-rules of 'a='
    attribute =           (att-field ":" att-value) / att-field
  
    att-field =           token
  
    att-value =           byte-string
  
    ; sub-rules of 'm='
    media =               token
                          ;typically "audio", "video", "text", or
                          ;"application"
  
    fmt =                 token
                          ;typically an RTP payload type for audio
                          ;and video media
  
    proto  =              token *("/" token)
                          ;typically "RTP/AVP" or "udp"
  
    port =                1*DIGIT
  
    ; generic sub-rules: addressing
    unicast-address =     IP4-address / IP6-address / FQDN / extn-addr
  
    multicast-address =   IP4-multicast / IP6-multicast / FQDN
                          / extn-addr
  
    IP4-multicast =       m1 3( "." decimal-uchar )
                          "/" ttl [ "/" integer ]
                          ; IPv4 multicast addresses may be in the
                          ; range 224.0.0.0 to 239.255.255.255
  
    m1 =                  ("22" ("4"/"5"/"6"/"7"/"8"/"9")) /
                          ("23" DIGIT )
  
    IP6-multicast =       hexpart [ "/" integer ]
                          ; IPv6 address starting with FF
  
    ttl =                 (POS-DIGIT *2DIGIT) / "0"
  
    FQDN =                4*(alpha-numeric / "-" / ".")
                          ; fully qualified domain name as specified
                          ; in RFC 1035 (and updates)
    IP4-address =         b1 3("." decimal-uchar)
  
    b1 =                  decimal-uchar
                          ; less than "224"
  
    ; The following is consistent with RFC 2373 [30], Appendix B.
    IP6-address =         hexpart [ ":" IP4-address ]
  
    hexpart =             hexseq / hexseq "::" [ hexseq ] /
                          "::" [ hexseq ]
  
    hexseq  =             hex4 *( ":" hex4)
  
    hex4    =             1*4HEXDIG
  
    ; Generic for other address families
    extn-addr =           non-ws-string
  
    ; generic sub-rules: datatypes
    text =                byte-string
                          ;default is to interpret this as UTF8 text.
                          ;ISO 8859-1 requires "a=charset:ISO-8859-1"
                          ;session-level attribute to be used
  
    byte-string =         1*(%x01-09/%x0B-0C/%x0E-FF)
                          ;any byte except NUL, CR, or LF
  
    non-ws-string =       1*(VCHAR/%x80-FF)
                          ;string of visible characters
  
    token-char =          %x21 / %x23-27 / %x2A-2B / %x2D-2E / %x30-39
                          / %x41-5A / %x5E-7E
  
    token =               1*(token-char)
  
    email-safe =          %x01-09/%x0B-0C/%x0E-27/%x2A-3B/%x3D/%x3F-FF
                          ;any byte except NUL, CR, LF, or the quoting
                          ;characters ()<>
  
    integer =             POS-DIGIT *DIGIT
  
    ; generic sub-rules: primitives
    alpha-numeric =       ALPHA / DIGIT
  
    POS-DIGIT =           %x31-39 ; 1 - 9
    decimal-uchar =       DIGIT
                          / POS-DIGIT DIGIT
                          / ("1" 2*(DIGIT))
                          / ("2" ("0"/"1"/"2"/"3"/"4") DIGIT)
                          / ("2" "5" ("0"/"1"/"2"/"3"/"4"/"5"))
  
    ; external references:
          ; ALPHA, DIGIT, CRLF, SP, VCHAR: from RFC 4234
          ; URI-reference: from RFC 3986
          ; addr-spec: from RFC 2822

# 12. 参考

## 12.1.  Normative References

    [1]   Mockapetris, P., "Domain names - concepts and facilities", STD
          13, RFC 1034, November 1987.
  
    [2]   Mockapetris, P., "Domain names - implementation and
          specification", STD 13, RFC 1035, November 1987.
  
    [3]   Bradner, S., "Key words for use in RFCs to Indicate Requirement
          Levels", BCP 14, RFC 2119, March 1997.
  
    [4]   Crocker, D., Ed. and P. Overell, "Augmented BNF for Syntax
          Specifications: ABNF", RFC 4234, October 2005.
  
    [5]   Yergeau, F., "UTF-8, a transformation format of ISO 10646", STD
          63, RFC 3629, November 2003.
  
    [6]   Handley, M. and V. Jacobson, "SDP: Session Description
          Protocol", RFC 2327, April 1998.
  
    [7]   Berners-Lee, T., Fielding, R., and L. Masinter, "Uniform
          Resource Identifier (URI): Generic Syntax", STD 66, RFC 3986,
          January 2005.
  
    [8]   Narten, T. and H. Alvestrand, "Guidelines for Writing an IANA
          Considerations Section in RFCs", BCP 26, RFC 2434, October
          1998.
  
    [9]   Alvestrand, H., "Tags for the Identification of Languages", BCP
          47, RFC 3066, January 2001.
  
    [10]  Olson, S., Camarillo, G., and A. Roach, "Support for IPv6 in
          Session Description Protocol (SDP)", RFC 3266, June 2002.
  
    [11]  Faltstrom, P., Hoffman, P., and A. Costello,
          "Internationalizing Domain Names in Applications (IDNA)", RFC
          3490, March 2003.
  
    [12]  Josefsson, S., "The Base16, Base32, and Base64 Data Encodings",
          RFC 3548, July 2003.

## 12.2.  Informative References

    [13]  Mills, D., "Network Time Protocol (Version 3) Specification,
          Implementation", RFC 1305, March 1992.
  
    [14]  Handley, M., Perkins, C., and E. Whelan, "Session Announcement
          Protocol", RFC 2974, October 2000.
  
    [15]  Rosenberg, J., Schulzrinne, H., Camarillo, G., Johnston, A.,
          Peterson, J., Sparks, R., Handley, M., and E. Schooler, "SIP:
          Session Initiation Protocol", RFC 3261, June 2002.
  
    [16]  Schulzrinne, H., Rao, A., and R. Lanphier, "Real Time Streaming
          Protocol (RTSP)", RFC 2326, April 1998.
  
    [17]  Rosenberg, J. and H. Schulzrinne, "An Offer/Answer Model with
          Session Description Protocol (SDP)", RFC 3264, June 2002.
  
    [18]  Camarillo, G., Eriksson, G., Holler, J., and H. Schulzrinne,
          "Grouping of Media Lines in the Session Description Protocol
          (SDP)", RFC 3388, December 2002.
  
    [19]  Schulzrinne, H., Casner, S., Frederick, R., and V. Jacobson,
          "RTP: A Transport Protocol for Real-Time Applications", STD 64,
          RFC 3550, July 2003.
  
    [20]  Schulzrinne, H. and S. Casner, "RTP Profile for Audio and Video
          Conferences with Minimal Control", STD 65, RFC 3551, July 2003.
  
    [21]  Casner, S., "Session Description Protocol (SDP) Bandwidth
          Modifiers for RTP Control Protocol (RTCP) Bandwidth", RFC 3556,
          July 2003.
  
    [22]  Huitema, C., "Real Time Control Protocol (RTCP) attribute in
          Session Description Protocol (SDP)", RFC 3605, October 2003.
  
    [23]  Baugher, M., McGrew, D., Naslund, M., Carrara, E., and K.
          Norrman, "The Secure Real-time Transport Protocol (SRTP)", RFC
          3711, March 2004.
    [24]  Rosenberg, J., Schulzrinne, H., and P. Kyzivat, "Indicating
          User Agent Capabilities in the Session Initiation Protocol
          (SIP)", RFC 3840, August 2004.
  
    [25]  Westerlund, M., "A Transport Independent Bandwidth Modifier for
          the Session Description Protocol (SDP)", RFC 3890, September
          2004.
  
    [26]  International Telecommunication Union, "H.323 extended for
          loosely coupled conferences", ITU Recommendation H.332,
          September 1998.
  
    [27]  Arkko, J., Carrara, E., Lindholm, F., Naslund, M., and K.
          Norrman, "Key Management Extensions for Session Description
          Protocol (SDP) and Real Time Streaming Protocol (RTSP)", RFC
          4567, July 2006.
  
    [28]  Andreasen, F., Baugher, M., and D. Wing, "Session Description
          Protocol (SDP) Security Descriptions for Media Streams", RFC
          4568, July 2006.
  
    [29]  Resnick, P., "Internet Message Format", RFC 2822, April 2001.
  
    [30]  Hinden, R. and S. Deering, "IP Version 6 Addressing
          Architecture", RFC 2373, July 1998.
  
    [31]  Freed, N. and J. Klensin, "Media Type Specifications and
          Registration Procedures", BCP 13, RFC 4288, December 2005.
