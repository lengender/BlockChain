﻿# Merkle Tree

---

这段时间学习区块链相关知识，接触到了Merkle Tree的概念，总结一下。

### Hash List
在点对点网络中作数据传输的时候，会同时从多个机器上下载数据，而且很多机器可以认为是不稳定或者不可信的。为了校验数据的完整性，更好的方法是把大的文件分割成小的数据块。这样的好处是，如果小数据块在传输过程中损坏了，那么只要重新下载这一块数据就可以了，不用重新下载整个文件。

怎么确定小的数据块没有损坏呢？ 只要为每个数据块做Hash。BT下载的时候，在下载到真正数据之前，我们会先现在一个Hash列表。那么问题来了，怎么确定这个Hash列表本身是正确地呢？答案是把每个小块数据的Hash值拼到一起，然后对这个长字符串再作一次Hash运算，这样就得到了Hash列表的根Hash（Top Hash或Root Hash）。下载数据的时候，首先从可信的数据源得到正确的根Hash，就可以用它来校验Hash列表了，然后通过校验后的Hash列表校验数据块。

<div align = "center">
<img src="https://raw.githubusercontent.com/lengender/MarkdownPhotos/master/HashList.png" width="500" height="400"/>
    </div>
    
    
### 什么是Merkle Tree
Merkle Tree可以看做Hash List的泛化。(Hash List可以看作一种特殊的Merkle Tree, 即树高为2的多叉Merkle Tree)

在最底层，和哈希列表一样，我们把数据分为小的数据块，有相应的哈希和它对应。但是往上走，并不直接去运算根哈希，而是把相邻的两个哈希合并成一个字符串，然后运算得到这个字符串的哈希，这样每两个哈希就结婚生子，得到了一个“子哈希”。如果最底层的哈希总数是单数，那到最后必然出现一个单身哈希，这种情况就直接对它进行哈希运算，所以也能得到子哈希。于是往上推，依然是一样的方式，可以得到数目更少的新一级哈希，最终必然形成一颗倒挂的树，到了树根这个位置，这一代就剩下一个根哈希了，我们叫它为Merkle root.

Merkle Tree是一种树，由于其所有的节点都是hash值，也称为Merkle Hash Tree。具有以下特点：
1. 它是一种树，可以是二叉树，也可以是多叉树，无论是几叉树，它都具有树结构的所有特点。
2. Merkle树的叶子节点上的value，是由你指定的，这主要看你的设计了。如Merkle Hash Tree会将数据的Hash值作为叶子节点的值。
3. 非叶子节点的value是根据它下面所有的叶子节点值，然后按照一定得算法计算得到的。如Merkle Hash Tree的非叶子节点value的计算方法是将该节点的所有子节点进行组合，然后对组合结果进行hash计算所得到的hash value.

## Merkle Tree的应用
### 数字签名
最初Merkle Tree目的是高效的处理lamport one-time signatures。每一个 lamport key只能被用来签名一个消息，但是与Merkle tree结合可以来签名多条Merkel。这种方法成为了一种高效的数字签名框架，即Merkle Signature Scheme。

### P2P网络
在P2P网络中，Merkle Tree用来确保从其他节点接受的数据块没有被损坏和替换，甚至检查其他节点会不会欺骗或者发布虚假的块。大家所熟悉的BT下载就是采用了P2P技术来让客户端之间进行数据传输，一来可以加快下载速度，二来减轻下载服务器的负担。BT即BitTorrent，是一种中心索引式的P2P文件分析通信协议。

开始下载时，必须从中心索引服务器获取一个扩展名为torrent的索引文件，torrent文件包含了要共享文件的信息，包括文件名，大小，文件的hash信息和一个指向tracker的url。Torrent文件中的hash信息是每一块要下载的文件内容的加密摘要，这些摘要也可运行在下载的时候进行验证。大的torrent文件是web服务器的瓶颈，而且也不能直接被包含在RSS或gossiped around(用流言传播协议进行传播)。一个相关的问题是大数据块的使用，因为为了保持torrent文件的非常小，那么数据块hash的数量也得很小，这就意味着每个数据块相对较大。大数据库影响节点之间进行交易的效率，因为只有当大数据块全部下载下来并校验通过后，才能与其他节点进行交易。

### Trusted Computing
可信计算是可信计算组为分布式计算环境中参与节点的计算平台提供端点可信性而提出的。可信计算技术在计算平台的硬件层引入可信平台模块(Trusted Platform，TPM)，实际上为计算平台提供了基于硬件的可信根(Root of Trust, RoT)。从可信根出发，使用信任链传递机制，可信计算技术可对本地平台的硬件及软件实施逐层的完整性度量，并将度量结果可靠地保存在TPM的平台配置寄存器(Platform configuration register, PCR)中，此后远程计算平台通过远程验证机制(Remote Attestation)比对本地PCR中度量结果，从而验证本地计算平台的可信性。

可信计算技术让分布式应用的参与节点摆脱了对中心服务器的依赖，而直接通过用户机器上的TPM芯片来建立信任，是的创建扩张性更好，可靠性更高，可用性更强的安全分布式应用成为可能。可信计算技术的核心机制是远程验证(remote attestation)，分布式应用的参与节点正是通过远程验证机制来建立互信，从而保障应用的安全。

### IPFS
IPFS(InterPlanetary File System)是很多NB的互联网技术的综合体，例如DHT(Distributed HashTable，分布式哈希表)，Git版本控制系统，Bittorrent等。它创建了一个P2P的集群，这个集群允许IPFS对象的交换。全部的IPFS对象形成了一个被称为Merkle DAG的加密认证数据结构。

IPFS对象是一个含有两个域的数据结构：

* Data - 非结构的二进制数据，大小小于256kB
* Links - 一个Link数据结构的数组，IPFS对象通过他们链接到其他对象

Link数据结构包含三个域：

* Name - Link的名字
* Hash - Link链接到对象的Hash
* Size - Link链接到对象的累积大小，包括它的Links

### BitCoin和Ethereum
Merkle Proof最早的应用是BitCoin，它是由中本聪在2009年描述并创建的。BitCoin的Blockchain利用Merkle Proofs来存储每个区块的交易。

<div align = "center">
<img src="https://raw.githubusercontent.com/lengender/MarkdownPhotos/master/MerkleProofs.png" width="500" height="400"/>
    </div>

而这样做的好处，也就是中本聪描述到的“简化支付验证”(Simplified Payment Verification, SPV)的概念：一个轻客户端(light client)可以仅下载链的区块头即每个区块中的80byte的数据块，仅包含五个元素，而不是下载每一笔交易以及每一个区块。

* 上一个区块的哈希头
* 时间戳
* 挖矿难度值
* 工作量证明随机数(Nonce)
* 包含该区块交易的Merkle Tree的根哈希

如果客户端想要确认一个交易的状态，它只需简单的发起一个Merkle Proof请求，这个请求显示出这个特定的交易在Merkle trees的一个之中，而且这个Merkle Tree的树根在主链的一个区块头中。

但是BitCoin的轻客户端有它的局限。一个局限是，尽管它可以证明包含的交易，但是它不能进行涉及当前状态的证明(如数字资产的持有，名称注册，金融合约的状态等)

Bitcoin如何查询你当前有多少个币？ 一个比特币轻客户端，可以使用一种协议，它涉及查询多个节点，并相信其中至少会有一个节点会通知你，关于你的地址中任何特定的交易支出，而这可以让你实现更多的应用。但是对于其他更为复杂的应用而言，这些远远是不够的。一笔交易影响的确切性质(precise nature)，可以取决于此前的几笔交易，而这些交易本身则依赖于更为前面的交易，所以最终你可以验证整个链上的每一笔交易。为了解决这个问题，Ethereum的Merkle Tree的概念，会更进一步。

### Ethereum的Merkle Proof
每个以太坊区块头不是包括一个Merkle树，而是为三种对象设计的三棵树：

* 交易Transaction
* 收据Receipts(本质上是显示每个交易影响的多块数据)
* 状态State

<div align = "center">
<img src="https://raw.githubusercontent.com/lengender/MarkdownPhotos/master/eth_%E5%8C%BA%E5%9D%97.png" width="500" height="400"/>
    </div>

这使得一个非常先进的轻客户端协议成为了可能，它允许轻客户端轻松地进行并核实以下类型的查询答案：

* 这笔交易被包含在特定的区块中了吗？
* 告诉我这个地址在过去30天中，发出X类型事假的所有实例(例如，一个众筹合约完成了它的目标)
* 目前我的账户余额是多少？
* 这个账户是否存在？
* 假如在这个合约中运行这笔交易，它的输出会是什么？

第一种是由交易树(transaction tree)来处理的；第三种和第四种则是由状态树(state tree)负责处理，第二种则由收据树(receipt tree)处理。计算前四个查询任务是相当简单的。服务器简单地找到对象，获取Merkle分支，并通过分支来回复轻客户端。

第五种查询任务同样也是由状态树处理，但它的计算方式会比较复杂。这里，我们需要构建一个Merkle状态转变证明(Merkle state transition proof)。从本质上讲，这样的证明也就是说“如果你在根S的状态树上运行交易T，其结果状态树将是根为S'，log为L，输出为O”（“输出”作为存在于以太坊的一种概念，因为每一笔交易都是一个函数调用；它在理论上并不是必要的）

为了推断这个证明，服务器在本地创建了一个假的区块，将状态设为S，并在请求这笔交易时假装是一个轻客户端。也就是说，如果请求这笔交易的过程，需要客户端确定一个账户的余额，这个轻客户端(由服务器模拟的)会发出一个余额查询请求。如果需要轻客户端在特定某个合约的存储中查询特定的条目，这个轻客户端就会发出这样的请求。也就是说服务器(通过模拟一个轻客户端)正确回应所有自己的请求，但服务器也会跟踪它发回的所有数据。

然后，服务器从上述的这些请求中把数据合并并把数据以一个证明的方式发送给客户端。

然后，客户端会进行相同的步骤，但会将服务器提供的证明作为一个数据库来使用。如果客户端进行步骤的结果和服务器提供的是一样的话，客户端就接受这个证明。

## 参考
[Merkle Tree学习][1]


  [1]: https://www.cnblogs.com/fengzhiwu/p/5524324.html