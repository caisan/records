---
layout: post
title: 简单理解磁盘结构(转)
tags:
- LinuxOps
categories: linuxOps
description: 简单理解磁盘结构
---


数据库系统总会涉及到辅助存储（大多都是磁盘），因为它们能够存储大量需要长期保存的数据，因此我们有必要先了解了解磁盘的相关知识。

根据机械原理，存储器的容量越大其速度就越慢。但是速度越快的存储器，其单位字节的价格就越贵。现代计算机系统可以包含几个不同的可以存储数据的部件，就形成了存储器的层次结构，但是需要注意的是「虚拟内存」是操作系统与操作系统运用机器硬件的产物，它不是存储器的层次之一。



<!-- more -->


## 1. 磁盘结构

传统的硬盘盘结构是像下面这个样子的，它有一个或多个盘片，用于存储数据。盘片多采用铝合金材料；中间有一个主轴，所有的盘片都绕着这个主轴转动。一个组合臂上面有多个磁头臂，每个磁头臂上面都有一个磁头，负责读写数据。

![linuxops-disk-details1](https://ivanzz1001.github.io/records/assets/img/linux/linuxops_disk_details1.jpg)

磁盘一般有一个或多个盘片。每个盘片可以有两面，即第一个盘片的正面为0面，反面为 1 面；第二个盘片的正面为 2 面…依次类推。磁头的编号也和盘面的编号是一样的，因此有多少个盘面就有多少个磁头。盘面正视图如下图，磁头的传动臂只能在盘片的内外磁道之间移动。因此不管开机还是关机，磁头总是在盘片上面。关机时，磁头停在盘片上面，抖动容易划伤盘面造成数据损失，为了避免这样的情况，所以磁头都是停留在起停区的，起停区是没有数据的。

![linuxops-disk-details2](https://ivanzz1001.github.io/records/assets/img/linux/linuxops_disk_details2.jpg)

每个盘片的盘面被划分成多个狭窄的同心圆环，数据就存储在这样的同心圆环上面，我们将这样的圆环称为```磁道``` (Track)。每个盘面可以划分多个磁道，最外圈的磁道是0号磁道，向圆心增长依次为1磁道、2磁道…磁盘的数据存放就是从最外圈开始的。

![linuxops-disk-details3](https://ivanzz1001.github.io/records/assets/img/linux/linuxops_disk_details3.jpg)

根据硬盘的规格不同，磁道数可以从几百到成千上万不等。每个磁道可以存储数 Kb 的数据，但是计算机不必要每次都读写这么多数据。因此，再把每个磁道划分为若干个弧段，每个弧段就是一个扇区 (Sector)。扇区是硬盘上存储的物理单位，现在每个扇区可存储 512 字节数据已经成了业界的约定。也就是说，即使计算机只需要某一个字节的数据，但是也得把这个 512 个字节的数据全部读入内存，再选择所需要的那个字节。

![linuxops-disk-details4](https://ivanzz1001.github.io/records/assets/img/linux/linuxops_disk_details4.jpg)

```柱面```是我们抽象出来的一个逻辑概念，简单来说就是处于同一个垂直区域的磁道称为柱面 ，即各盘面上面相同位置磁道的集合。需要注意的是，磁盘读写数据是按柱面进行的，磁头读写数据时首先在同一柱面内从 0 磁头开始进行操作，依次向下在同一柱面的不同盘面(即磁头上)进行操作，只有在同一柱面所有的磁头全部读写完毕后磁头才转移到下一柱面。因为选取磁头只需通过电子切换即可，而选取柱面则必须通过机械切换。数据的读写是按柱面进行的，而不是按盘面进行，所以把数据存到同一个柱面是很有价值的。

磁盘被磁盘控制器所控制（可控制一个或多个），它是一个小处理器，可以完成一些特定的工作。比如将磁头定位到一个特定的半径位置；从磁头所在的柱面选择一个扇区；读取数据等。

![linuxops-disk-details5](https://ivanzz1001.github.io/records/assets/img/linux/linuxops_disk_details5.jpg)

现代硬盘寻道都是采用CHS(Cylinder Head Sector)的方式，硬盘读取数据时，读写磁头沿径向移动，移到要读取的扇区所在磁道的上方，这段时间称为寻道时间(seek time)。因读写磁头的起始位置与目标位置之间的距离不同，寻道时间也不同。磁头到达指定磁道后，然后通过盘片的旋转，使得要读取的扇区转到读写磁头的下方，这段时间称为旋转延迟时间(rotational latencytime)。然后再读写数据，读写数据也需要时间，这段时间称为传输时间（transfer time）。

根据上文的信息，我们可以得出磁盘容量的计算公式为：
<pre>
硬盘容量 = 盘面数 × 柱面数 × 扇区数 × 512字节
</pre>

## 2. 笔试题实战
下面的题目是腾讯某一年校招笔试中的一个题目，题干信息描述为：数据存储在磁盘上的排列方式会影响I/O服务的性能，一个圆环磁道上有10个物理块，10个数据记录R1~R10存放在这个磁道上，记录的安排顺序如下表所示。
<pre>
物理块	1	2	3	4	5	6	7	8	9	10
逻辑记录	R1	R2	R3	R4	R5	R6	R7	R8	R9	R10
</pre>
假设磁盘的旋转速度为20ms，磁盘当前处在R1的开头处，若系统顺序扫描后将数据放入单缓冲区内，处理数据的时间为4ms（然后再读取下个记录），则处理这10个记录的最长时间是多少？

答案：磁盘会一直朝某个方向旋转，不会因为处理数据而停止。本题要求顺序处理 R1 到 R10，起始位置在 R1，一周是 20ms，共 10 个记录，所以每个记录的读取时间为 2ms。首先读 R1 并处理 R1，读 R1 花 2ms，读好后磁盘处于 R1 的末尾或 R2 的开头，此时处理 R1，需要 4ms，因为磁盘一直旋转，所以 R1 处理好了后磁盘已经转到 R4 的开始了，这时花的时间为 2+4=6ms。这时候要处理 R2，需要等待磁盘从 R5 一直转到 R2 的开始才行，磁盘转动不可反向，所以要经过 ```8*2ms``` 才能转到 R1 的末尾，读取 R2 需要 2ms，再处理 R2 需要 4ms，处理结束后磁盘已经转到 R5 的开头了，这时花的时间为 ```2*8+2+4=22ms```。等待磁盘再转到 R3 又要 ```8 * 2ms```，加上 R3 自身 2ms 的读取时间和 4ms 的处理时间，花的时间也为 22ms，此时磁盘已经转到 R6 的开头了，写到这里，就可以看到规律了，读取并处理后序记录都为 22ms，所以总时间为
```6+22*9=204ms```。


## 3. 如何加速对磁盘的访问

对于理解数据库系统系统特别重要的是磁盘被划分为磁盘块（或像操作系统一样称之为页），每个块的大小是 4~64KB。磁盘访问一个磁盘块平均要用 10ms，但是这并不表示某一应用程序将数据请求发送到磁盘控制器后，需要等 10ms 才能得到数据。如果只有一个磁盘，在最坏的情况下，磁盘访问请求的到达个数超过 10ms 一次，那么这些请求就会被无限的阻塞，调度延迟将会变的非常大。因此，我们有必要做一些事情来减少磁盘的平均访问时间。

按柱面组织数据：前这一点在前文已经提到过了。因为寻道时间占平均块访问时间的一半，如果我们选择在一个柱面上连续的读取所有块，那么我们只需要考虑一次寻道时间，而忽略其它时间。这样，从磁盘上读写数据的速度就接近于理论上的传输速率。

使用多个磁盘：如果我们使用多个磁盘来替代一个磁盘，只要磁盘控制器、总线和内存能以 n 倍速率处理数据传输，则使用 n 个磁盘的效果近似于 1 个磁盘执行了 n 次操作。因此使用多个磁盘可以提高系统的性能。

**磁盘调度**：提高磁盘系统吞吐率的另一个有效方法是让磁盘控制器在若干个请求中选择一个来首先执行，调度大量块请求的一个简单而有效的方法就是电梯算法。回忆一下电梯的运行方式，它并不是严格按先来后到的顺序为乘客服务，而是从建筑物的底层到顶层，然后再返回来。同样，我们把磁盘看作是在做横跨磁盘的扫描，从柱面最内圈到最外圈，然后再返回来，正如电梯做垂直运动一样。

**预取数据**：在一些应用中，我们是可以预测从磁盘请求块的顺序的。因此我们就可以在需要这些块之前就将它们装入主存。这样做的好处是我们能较好的调度磁盘，比如采用前文的电梯算法来减少访问块所需要的平均时间。

## 4. 磁盘故障
如果事情都像我们一开始设计的那样进行，那世界肯定会变得特别无聊。磁盘偶尔也会耍耍小脾气，甚至是罢工不干了。比如在读写某个扇区一次尝试没有成功，但是反复尝试后有成功读写了，我们称之为间歇性故障。

一种更为严重的故障形式是，一个或多个二进制位永久的损坏了，所以不管我们尝试多少次都不可能成功，这种故障称之为介质损坏。

另一种相关的错误类型称之为写故障，当我们企图写一个扇区时，既不能正确的写，也不能检索先前写入的扇区，发生这种情况的一种可能原因就是在写过程中断电了。

当然肯定最严重的就是磁盘崩溃，这种故障中，整个磁盘都变为永久不可读，这是多么可怕的事情。

既然会出现上面所述的各种大小故障，那么我们就必须要采取各种措施去应对大大小小的变故，保证系统能正常运行。

## 5. 规避故障
我们尝试读一个磁盘块，但是该磁盘块的正确内容没有被传送到磁盘控制器中，就是一个间歇性故障发生了。那么问题是控制器如何能判断传入的内容是否正确呢？答案就是使用校验和，即在每个扇区使用若干个附加位。在读出时如果我们发现校验和对数据位不合适，那么我们就知道有错误；如果校验和正确，磁盘读取仍然有很小的可能是不正确的，但是我们可以通过增加趣多校验位来降低读取不正确发生的概率。

此处我们使用奇偶校验来举例，通过设置一个校验位使得二进制集合中 1 的个数总是偶数。比如某个扇区的二进制位序列是 01101000，那么就有奇数个 1，所以奇偶位是 1，这个序列加上它后面的奇偶位，就有 011010001；而如果所给的序列是 11101110，那么奇偶位就是 0。所以每一个加上了奇偶位构成的 9 位序列都有偶数奇偶性。

尽管校验和几乎能正确检测出介质故障或读写故障的存在，但是它却不能帮助我们纠正错误。为了处理这个问题，我们可以在一个或多个磁盘中执行一个被称为稳定存储的策略。通常的思想是，扇区时成对的，每一对代表一个扇区内容 X。我们把代表 X 的扇区对分别称为左拷贝 XL和右拷贝XR。这样实际上就是每个扇区的内容都存储了两份，操作XL失败，那么去操作XR就可以了，更何况我们还在每个扇区中有校验和，把错误的概率就大大降低了。

到现在为止，我们讨论的都是简单的故障，但是如果发生了磁盘崩溃，其中的数据被永久破坏。而且数据没有备份到另一种介质中，对于银行金融系统这将是巨大的灾难，遇到这种情况我们应该怎么办呢？

## 6. 数据恢复
应对磁盘故障最简单的方式就是镜像磁盘，即我们常说的备份。回忆一下写毕业论文时的做法，那时候大部分同学还不会用版本控制器，所以基本采用每天备份一次数据，并且在文件名称中标注日期，以此来达到备份的效果。

第二种方式是使用奇偶块，比如一个系统中有 3 个磁盘，那么我们再加一个磁盘作为冗余盘。在冗余盘中，第 i 块由所有数据盘的第 i 块奇偶校验位组成。也就是说，所有第 I 块的第 j 位，包括数据盘和冗余盘，在它们中间必须有偶数个 1，冗余盘的作用就是让这个条件为真。

我们举个简单例子，假设快仅由一个字节组成，我们有三个数据盘和一个冗余盘，对应的位序列如下。其中 盘4 为冗余盘，它的位序列是根据前面三个盘计算出来的。
<pre>
盘 1：11110000
盘 2：10101010
盘 3：00111000
盘 4：01100010
</pre>
假设现在某个盘崩溃了，那么我们就能根据上面的序列来恢复数据，只需要让每一列 1 的个数为偶数就可以了，但是这种冗余方式也存在很大的不足。

第一个缺陷是，如果是两个盘同时崩溃了，那数据也恢复不出来了。第二个问题在于，虽然读数据只需要一次 I/O 操作即可，但是写数据时就不一样了，因为需要根据其他数据盘来计算冗余盘中的位序列，假设共有 n 个盘，其中一个为冗余盘，所以每次写数据时，都需要进行 n+1 次 I/O 操作（读不被写入的 n-1 个盘，被重写数据盘的一次写，冗余盘的一次写），而 I/O操作又是非常耗时的操作，所以这种方法会大大拖慢系统性能。

另一种方案是没有明显的冗余盘，而是把每个磁盘作为某些块的冗余盘来处理。比如现在有 4 个盘，0 号磁盘将作为编号为 4、8、12 等柱面的冗余，而 1 号磁盘作为编号为 1、5、9 等块的冗余…

一种更为先进的方式使用海明码来帮助从故障中恢复数据，它在多个磁盘崩溃的情况下也能恢复出数据，也是 RAID 的最高等级，由于本人水平有限，用文字表达不清楚，就不作介绍了，嘿嘿。


<br />
<br />

**[参看]:**

1. [简单理解磁盘结构](https://blog.csdn.net/heuguangxu/article/details/80072024)

<br />
<br />
<br />

