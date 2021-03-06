# 第三章：数据链路层

## 1. 数据链路层的研究思想

对于五层模型来说，数据的传输从上到下，再从下到上的过程，即：封装数据，解封装数据的过程

![](.gitbook/assets/image%20%2836%29.png)

我们研究数据链路层，把数据看做是水平传输，仅仅研究数据链路层 - 数据链路层

## 2. 数据链路层基本概念

**结点：**主机、路由器

**链路（link）：**从一个结点到**相邻结点**的一段物理路线，而中间没有任何其他交换结点。

**数据链路（data link）：**数据在一条路线上传输时，除了必须有的一条物理路线外，还必须有一些通信协议来控制这些数据的传输。若**把实现这些协议的硬件和软件加到链路连上**，就构成了数据链路。

**帧：**数据链路层把网络层交下来的数据，添加自己的首部和尾部，构成帧，传输出去。

**数据链路层的功能：**负责通过一条链路从一个结点向另一个物理链路直接相连的相邻结点传送数据报。

## 3. 数据链路层功能概述

数据链路层在物理层提供服务的基础上**向网络层提供服务**，其最基本的服务是将源自网络层来的数据**可靠**地传输到相邻节点的目标机网络层。

其主要作用是**加强物理层传输原始比特流的功能**，将物理层提供的可能出错的物理连接改造为**逻辑上无差错的数据链路**，使之对网络层表现为一条无差错的链路。

* **功能一：**为网络层提供服务。**无确认无连接服务**、**有确认无连接服务**、**有确认面向连接服务（有连接一定有确认）**。
* **功能二：**链路管理，即连接的建立、维持、释放（用于面向连接的服务）。
* **功能三：**组帧
* **功能四：**流量控制
* **功能五：**差错控制（帧错 / 位错）

### 3.1 封装成帧 & 透明传输

**封装成帧（framing）**就是在一段数据的前后分别添加首部和尾部，这样就构成了一个帧。接收端在收到物理层上交的比特流后，就能根据首部和尾部的标记，从收到的比特流中识别出帧的开始和结束。

首部和尾部包含许多的控制信息，它们的一个重要作用：**帧定界**（确定帧的界限）。

**帧同步：接收方**应当能从接受到的二进制比特流中区分出帧的起始和终止。

**最大传送单元 MTU**（Maximum Transfer Unit）：规定帧的**数据部分的长度上限**

**组帧的四种方法：**1. 字符计数法，2. 字符（节）填充法，3. 零比特填充法，4. 违规编码法

![](.gitbook/assets/image%20%2847%29.png)

**透明传输：**指不管所传数据是什么样的比特组合，都应当能够在链路上传送。因此，链路就 “看不见” 有什么妨碍数据传输的东西。

当所传数据中的比特组合恰巧与某一个控制信息完全一样时，就必须采取适当的措施，使接收方不会将这样的数据误认为是某种控制信息。这样才能保证数据链路层的传输是透明的。

#### 1. 字符计数法（不常用）

帧首部使用一个计数字段（第一个**字节**，八位）来标明帧内字符数。

![](.gitbook/assets/image%20%2838%29.png)

**缺点：**如果第一个字节出现了错误，就会导致整个帧判断错误

#### 2. 字符填充法

帧的首部用 SOH（Start Of Header）表示，尾部用 EOT（End Of Transmission）表示。如图所示：

![](.gitbook/assets/image%20%2829%29.png)

**使用情况：**

1. ![](.gitbook/assets/image%20%2839%29.png) 当传送的帧是由文本文件组成时（文本文件的字符都是从键盘上输入的，都是 ASCII 码）。不管从键盘上输入什么字符都可以放在帧里传过去，即**透明传输**。
2. ![](.gitbook/assets/image%20%2854%29.png) 当传送的帧是由非 ASCII 码的文本组成时（二进制代码的程序或图像等）。就要**采用字符填充方法实现透明传输**。
   * 当帧中出现控制字符 “SOH” 或 “EOT” 的前面插入一个**转义字符 “ESC”**（其十六进制编码是 1B，二进制是 0001 1011）
   * 在**接收端**的数据链路层在把数据送往网络层之前删除这个插入的转义字符
   * 如果转义字符也出现在数据部分中，解决方法是在转义字符前面插入一个转义字符

![](.gitbook/assets/image%20%2828%29.png)

#### 3. 零比特填充法

在首部和尾部均用 0x7E（二进制：0111 1110）表示

![](.gitbook/assets/image%20%2833%29.png)

**操作：**

1. 在发送端，扫描整个信息字段，只要连续 5 个 1，就立即填入 1 个 0

   ![](.gitbook/assets/image%20%2851%29.png) 

2. 在接收端收到一个帧时，先找到标志字段确定边界，再用硬件对比特流进行扫描。发现连续 5 个 1 时，就把后面的 0 删除。

   ![](.gitbook/assets/image%20%2846%29.png) 

**保证了透明传输：在传送的比特流中可以传送任意比特组合，而不会引起对帧边界的判断错误。**

#### 4. 违规编码法

曼彻斯特编码： ![](.gitbook/assets/image%20%2848%29.png) 

由于曼彻斯特编码只会出现 **高-低**、**低-高** 的情况，所以我们可以使用 **高-高**，**底-底** 来定界帧的起始和终止。

**总结：**由于字节计数法中 Count 字段的脆弱性（其值若有差错将导致灾难性后果）及字符填充法实现上的复杂性和不兼容新，目前较普遍使用的帧同步是**比特填充**（零比特填充）和**违规编码法**。

### 3.2 差错控制（检错编码）

#### 差错从何而来？

概括的来说，传输中的差错都是由于**噪声引起**的。

* **（全局性）**由于线路本身电气特性所产生的的**随机噪声**（热噪声），是信道固有的，随机存在的。
  * **解决办法：**提高信噪比来减少或避免干扰。（对传感器下手）
* **（局部性）**外界特定的短暂原因所造成的**冲击噪声**，是产生差错的主要原因。
  * **解决办法：**通常利用编码技术来解决

**差错的分类：**

* **位错：**比特位出错，1 变为 0，0 变为 1
* **帧错：**
  * **帧丢失：**收到 \[\#1\]-\[\#3\]（丢失 \[\#2\]）
  * **帧重复：**收到 \[\#1\]-\[\#2\]-\[\#2\]-\[\#3\]（收到两个 \[\#2\]）
  * **帧失序：**收到 \[\#1\]-\[\#3\]-\[\#2\]

**数据链路层为网络层提供服务：**

* **无确认**无连接服务
  * 通信质量**好**的**有线**传输链路
  * 数据链路层**不**使用**确认**和**重传机制**，由上层传输层来解决
* **有确认**无连接服务、**有确认**面向连接服务
  * 通信质量**差**的**无线**传输链路
  * 数据链路层**使用确认**和**重传机制**

## 1. 使用点对点信道的数据链路层

### 1.1 数据链路和帧

**链路（link）：**从一个结点到**相邻结点**的一段物理路线，而中间没有任何其他交换结点。

**数据链路（data link）：**数据在一条路线上传输时，除了必须有的一条物理路线外，还必须有一些通信协议来控制这些数据的传输。若**把实现这些协议的硬件和软件加到链路连上**，就构成了数据链路。

**帧：**数据链路层把网络层交下来的数据，添加自己的首部和尾部，构成帧，传输出去。

![](.gitbook/assets/image%20%2852%29.png)

点对点信道的数据链路层在进行通信时的主要步骤为：

1. 结点 A 的数据链路层把网络层交下来的 IP 数据报添加首部和尾部封装成帧
2. 结点 A 把封装好的帧发送给结点 B 的数据链路层
3. 若结点 B 的数据链路层收到的帧无差错，则从收到的帧中提取出 IP 数据报交给上面的网络层；否则丢弃这个帧

### 1.2 三个基本问题

数据链路层的协议有很多，但是三个基本问题是共同的。这三个基本问题是：**封装成帧、透明传输、差错检测**。

#### 1.2.1 封装成帧

**封装成帧（framing）**就是在一段数据的前后分别添加首部和尾部，这样就构成了一个帧。

首部和尾部的作用：**帧定界**（确定帧的界限）。接收端在收到物理层上交的比特流后，就可以根据首部和尾部的标记，识别出帧的开始和结束。

**最大传送单元 MTU**（Maximum Transfer Unit）：规定帧的**数据部分的长度上限**

![&#x7528;&#x5E27;&#x9996;&#x90E8;&#x548C;&#x5E27;&#x5C3E;&#x90E8;&#x5C01;&#x88C5;&#x6210;&#x5E27;](.gitbook/assets/image%20%2853%29.png)

**帧定界**

* 当数据是由可打印的 ASCII 码组成的文本文件时，帧定界可以使用特殊的**帧定界符**
* 如果接收端收到的帧没有发现帧结束符，说明该帧是不完整的，丢弃

如下图所示：首部用 SOH（Start Of Header）表示；尾部用 EOT（End Of Tail）表示

注意：SOH 和 EOT 并不是用六个字母表示的，仅仅是控制字符的名称，它们十六进制的编码分别是 01（二进制是 0000 0001）和 04（二进制是 0000 0100）。因为这两个控制字符是 ASCII 码中不可打印的，所以选用它们，这样可以避免和帧的数据部分发生冲突。

![&#x7528;&#x63A7;&#x5236;&#x5B57;&#x7B26;&#x8FDB;&#x884C;&#x5E27;&#x5B9A;&#x754C;&#x7684;&#x65B9;&#x6CD5;&#x4E3E;&#x4F8B;](.gitbook/assets/image%20%2835%29.png)

#### 1.2.2 透明传输

**透明：**某一个实际存在的事物看起来却好像不存在一样。

**在数据链路层的透明传送数据：**表示无论什么样的比特组合的数据，都能够按照原样没有差错地通过这个数据链路层。

对于封装成帧中的帧定界符的选取方法，仅仅针对帧的数据部分是可打印的 ASCII 码，但是并非所有的数据都是这样的。所以对一一个非 ASCII 码的数据时，会出现一种情况，数据部分刚好包含 SOH 或 EOT 这种控制字符。数据链路层会**错误地** “找到帧的边界”，把部分帧收下（误认为是完整的帧），而把剩下的那部分数据丢弃。如果所示：

![&#x6570;&#x636E;&#x90E8;&#x5206;&#x6070;&#x597D;&#x51FA;&#x73B0;&#x4E8E;EOT&#x4E00;&#x6837;&#x7684;&#x4EE3;&#x7801;](.gitbook/assets/image%20%2841%29.png)

解决上述问题的方法：**字节填充**（byte stuffing）或**字符填充**（character stuffing）

* **发送端**的数据链路层在数据中出现控制字符 “SOH” 或 “EOT” 的前面插入一个**转义字符 “ESC”**（其十六进制编码是 1B，二进制是 0001 1011）
* 在**接收端**的数据链路层在把数据送往网络层之前删除这个插入的转义字符
* 如果转义字符也出现在数据部分中，解决方法是在转义字符前面插入一个转义字符

![&#x7528;&#x5B57;&#x8282;&#x586B;&#x5145;&#x6CD5;&#x89E3;&#x51B3;&#x900F;&#x660E;&#x4F20;&#x8F93;&#x7684;&#x95EE;&#x9898;](.gitbook/assets/image%20%2840%29.png)

#### 1.2.3 差错检测

**比特差错：**现实的通信链路都不会是理想的，比特在传输的过程中可能产生差错，1 可能会变为 0，而 0 也可能变为 1。

数据链路层广泛使用 **循环冗余检验 CRC**的检查技术。

CRC 的原理：

* 假设一个分组的数据 M 为 k 个比特，除数为 P
* 对数据做模运算（M % P）
* 把模运算的结果加到 M 的后面

**帧检验序列 FCS：**为了进行检错而添加的冗余码（模运算的结果）

示例：M = 101001（k = 6），P = 1101

![](.gitbook/assets/image%20%2834%29.png) 

> 运算后，余数为 001，把这个数拼接到 M 后面，即 101001 001

接收端把接收到的数据以帧为单位进行 CRC 检验：把收到的每一个帧都除以同样的除数 P，然后检查得到的余数 R

* 若得到的余数 $$R = 0 $$ ，则判定这个帧没有差错，接受
* 若得到的余数 $$R \neq 0$$ ，则判定这个帧有差错，丢弃

**重要理解：**

* 在数据链路层若**仅仅**使用循环冗余检验 CRC 差错检测技术，只能做到对帧的**无差错接受**，即：**“凡是接收端数据链路层接受帧，我们就能非常接近于 1 的概率认为这些帧在传输过程中没有产生差错”**。
* 注意，数据链路层**仅仅**对帧进行**差错检测**，并不能提供**可靠传输**的服务。更具体的，差错检测，只能检测出比特差错，即：帧的数据部分的比特由1变为0，由0变为1。
* 还有一类更复杂的传输差错，数据链路层检测不出来，即：收到的帧没有出现比特差错，但是出现了**帧丢失、帧重复**或**帧失序**。如：发送三个帧：\[\#1\]-\[\#2\]-\[\#3\]
  * **帧丢失：**收到 \[\#1\]-\[\#3\]（丢失 \[\#2\]）
  * **帧重复：**收到 \[\#1\]-\[\#2\]-\[\#2\]-\[\#3\]（收到两个 \[\#2\]）
  * **帧失序：**收到 \[\#1\]-\[\#3\]-\[\#2\]

## 2. 点对点协议 PPP

### 2.1 PPP 协议的特点

互联网用户通常都要连接到某个 ISP 才能接入互联网。PPP 协议（Point-to-Point Protocol）就是用户计算机和 ISP 进行通信时所用的数据链路层协议。

![](.gitbook/assets/image%20%2845%29.png)

#### 2.1.1 PPP 协议应满足的需求

1. **简单**
   * IETF 在设计互联网体系结构时，就把最复杂的部分放到 TCP 协议中，而网际层协议 IP 相对比较简单，提供不可靠的数据报服务。所以数据链路层也没必要比 IP 协议有更多的功能，对于数据链路层的帧，不需要纠错，不需要序号，也不需要流量控制，仅仅把简单作为**首要要求**。
   * 总结：数据链路层的功能，接收方每收到一个帧，就进行 CRC 检验。如果 CRC 检验正确，就收下这个帧；反之，丢弃这个帧。
2. **封装成帧**
   * PPP 协议必须规定特殊的字符作为**帧定界符**，以便接收端可以准确的找到真的开始和结束位置
3. **透明性**
   * PPP 协议必须保证数据传输的透明性，即，如果数据中出现了和帧定界符一样的比特组合时，就要采取有效的措施来解决。
4. **多种网络层协议**
   * PPP 协议必须能够**在同一条物理链路上同时支持多种网络层协议**（如 IP 和 IPX等）的运行。当点对点链路所连接的是局域网或路由器时，PPP 协议必须同时支持在链路所连接的局域网或路由器上运行的各种网络层协议。
5. **多种类型链路**
   * 除了要支持多种网络层的协议外，PPP 还必须能够在多种类型的链路上运行。如：串行的（一次只发送一个比特）或并行的（一次并行发送多个比特），同步的或异步的，低速的或高速的，电的或光的，交换的（动态的）或非交换的（静态的）点对点链路。
6. **差错检测**
   * PPP 协议必须能够对接收端收到的帧进行检测，并**立即丢失有差错的帧**。这样可以节约网络资源，可以让错误的帧不在网络上继续转发。
7. **检测连接状态**
   * PPP 协议必须具有一种机制能够及时（不超过几分钟）自动检测出链路是否处于正常工作状态。当出现故障的链路隔了一段时间后又重新恢复正常工作时，就特别需要有这种及时检测功能。
8. **最大传送单元**
   * PPP 协议必须对每一种类型的点对点链路设置**最大传送单元 MTU** 的标准默认值。这样做是为了促进各种实现之间的互操作性。如果高层协议发送的分组过长并超过 MTU 的数值，PPP 就要丢弃这样的帧，并返回差错。
9. **网络层地址协商**
   * PPP 协议必须提供一种机制使通信的两个网络层的实体能够通过协商知道或能够配置彼此的网络层地址。因为如果仅仅在链路层建立了连接而不知道对方网络层地址，则还不能保证网络层可以传送分组。
10. **数据压缩协商**
    * PPP 协议必须提供一种方法来协商使用数据压缩算法。但 PPP 协议并不要求将数据压缩算法进行标准化。

#### 2.1.2 PPP 协议的组成

1. 一个将 IP 数据报封装到串行链路的方法
2. 一个用来建立、配置和测试数据链路连接的**链路控制协议 LCP**（Link Control Protocol）
3. 一套**网络控制协议 NCP**（Network Control Protocol）

### 2.2 PPP 协议的帧格式

#### 2.2.1 各字段的意义

![PPP &#x5E27;&#x7684;&#x683C;&#x5F0F;](.gitbook/assets/image%20%2842%29.png)

**首部（四个字段）：**

* 第一个字段是**标志字段 F**（Flag），规定为 0x7E，**表示一个帧的开始或结束**。连续两帧之间只需要用一个标志字段，如果出现两个连续的标志字段，就表示中间有一个空帧，应当丢弃。
* 第二个字段是**地址字段 A**，目前无实际含义
* 第三个字段是**控制字段 C**，目前无实际含义
* 第四个字段是 **2 字节的协议字段**
  * 0x0021，表示帧的信息字段是 IP 数据报
  * 0xC021，表示帧的信息字段是 LCP 的数据

**尾部（两个字段）：**

* 第一个字段是 使用 CRC 的帧检验序列 FCS
* 第二个字段和首部的第一个字段一样

#### 2.2.2 字节填充

当信息字段中出现了和标志字段一样的比特（0x7E）组合时，就必须采取一些措施使这种形式上和标志字段一样的比特组合不出现在信息字段中。具体填充方法如下：

* 把信息字段中出现的每一个 0x7E 字节转变成为 2 字节序列（0x7D，0x5E）
* 若信息字段中出现了一个 0x7D 的字节（即出现了和转义字符一样的比特组合），则把 0x7D 转变为 2 字节序列（0x7D，0x5D）
* 若信息字段中出现 ASCII 码的控制字符（即数值小于 0x20 的字符），则在该字符前面要加入一个 0x7D 字节，同时将该字符的编码加以改变。例如，出现 0x03，就要把它转变为 2 字节序列（0x7D，0x23）

#### 2.2.3 零比特填充

PPP 协议用在 SONET/SDH 链路时，使用同步传输（一连串的特比连续传送）而不是异步传输（逐个字符地传送）。在这种情况下，PPP 协议采用零比特填充方法来实现透明传输。

**具体做法：**

* 在信息字段中，只要发现有 5 个连续的 1，则立即填入一个 0
* 接收端，如果发现 5 个连续的 1，就把后面的 0 删除，还原数据

**原因：**

* 因为标志字段为 0x7E，二进制为：0111 1110，经过上述操作，就会将信息字段中出现标志字段的可能性避免

### 2.3 PPP 协议的工作状态



