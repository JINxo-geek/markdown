# Block chain

---

区块链技术是一个对多种技术的组合创新，多种技术包括：
+++

1、 共识算法:POW/POS/DPOS/PBFT/BFT-Raft/Paxos/Kafka

2、 P2P通讯:自举(bootstrapped)/连接/广播

3、 签名验签:ECDSA/secp256k1/ED25519/MultiSig 

4、 Hash锁定:Merkle树/MPT树 
+++

5、 UTXO记账:流水账 

6、 智能合约:P2PKH/P2SH/Oracle/状态机

7、 隐私保护:零知识证明、同态加密、CoinJoin加密技术

8、 私钥存储:HD协议(Hierarchical Deterministic Key Creation)、钱包Wallets、丢失找回

9、 算力分发：矿池分发 

---

## 拜占庭将军问题
---
工作量证明其实相当于提高了做叛徒（发布虚假区块）的成本，在工作量证明下，只有第一个完成证明的节点才能广播区块，竞争难度非常大，需要很高的算力，如果不成功其算力就白白的耗费了（算力是需要成本的），如果有这样的算力作为诚实的节点，同样也可以获得很大的收益（这就是矿工所作的工作），这也实际就不会有做叛徒的动机，整个系统也因此而更稳定

---

- https://anders.com/blockchain/hash.html 

  哈希算法会将一串字符串按照固定的算法生成一串固定长度的16进制字符串

Note:

256位bit也就是32位16进制

![100](http://oocfz31zv.bkt.clouddn.com/100.jpg)

  ![101](http://oocfz31zv.bkt.clouddn.com/101.jpg)

  每一个区块都包含一笔以上的交易。

---

```js
/*区块结构

第一个步骤是确定区块的结构。为了让事情尽可能简单，区块结构只包含最必要的部分：索引，时间戳，数据，散列值(hash)和前一个区块的散列值(hash)。*/
class Block {
    constructor(index, previousHash, timestamp, data, hash) {
        this.index = index;
        this.previousHash = previousHash.toString();
        this.timestamp = timestamp;
        this.data = data;
        this.hash = hash.toString();
    }
}
```

---

![img](https://ask.qcloudimg.com/draft/1046487/ofuu05m13o.png?imageView2//0/w/1620)

---

```js
//生成区块散列值区块需要散列值以保持数据的完整性。可以使用SHA-256算法生成这个值。
var calculateHash = (index, previousHash, timestamp, data) => {
    return CryptoJS.SHA256(index + previousHash + timestamp + data).toString();
};
/*生成一个区块

要生成一个区块，我们必须知道前一个块的散列值，并创建剩余所需内容（索引，散列，数据和时间戳）。其中区块数据是由用户提供的。*/
var generateNextBlock = (blockData) => {
    var previousBlock = getLatestBlock();
    var nextIndex = previousBlock.index + 1;
    var nextTimestamp = new Date().getTime() / 1000;
    var nextHash = calculateHash(nextIndex, previousBlock.hash, nextTimestamp, blockData);
    return new Block(nextIndex, previousBlock.hash, nextTimestamp, blockData, nextHash);
};
```

---



#### 生成区块散列值

区块需要散列值以保持数据的完整性。可以使用SHA-256算法生成这个值。应该指出，这个散列值与“ [挖掘](https://en.bitcoin.it/wiki/Mining) ” 无关，因为不需要去解决[工作证明](https://en.wikipedia.org/wiki/Proof-of-work_system)问题。

```js
var calculateHash = (index, previousHash, timestamp, data) => {
    return CryptoJS.SHA256(index + previousHash + timestamp + data).toString();
};
```



---



#### 生成一个区块

要生成一个区块，我们必须知道前一个块的散列值，并创建剩余所需内容（索引，散列，数据和时间戳）。其中区块数据是由用户提供的。

```js
var generateNextBlock = (blockData) => {
    var previousBlock = getLatestBlock();
    var nextIndex = previousBlock.index + 1;
    var nextTimestamp = new Date().getTime() / 1000;
    var nextHash = calculateHash(nextIndex, previousBlock.hash, nextTimestamp, blockData);
    return new Block(nextIndex, previousBlock.hash, nextTimestamp, blockData, nextHash);
};
```



---



#### 存储区块

使用Javascript数组在内存中存储区块链。区块链的第一个区块总是一个所谓的“创世纪区块”，内容是固定的。

```js
var getGenesisBlock = () => {
    return new Block(0, "0", 1465154705, "my genesis block!!", "816534932c2b7154836da6afc367695e6337db8a921823784c14378abed4f7d7");
};

var blockchain = [getGenesisBlock()];
```

---



#### 验证区块的完整性

我们必须能时刻验证区块或者区块链是否完整，尤其是当我们从其他节点接收到新块，需要决定是否接受它们的时候。

```js
var isValidNewBlock = (newBlock, previousBlock) => {
    if (previousBlock.index + 1 !== newBlock.index) {
        console.log('invalid index');
        return false;
    } else if (previousBlock.hash !== newBlock.previousHash) {
        console.log('invalid previoushash');
        return false;
    } else if (calculateHashForBlock(newBlock) !== newBlock.hash) {
        console.log('invalid hash: ' + calculateHashForBlock(newBlock) + ' ' + newBlock.hash);
        return false;
    }
    return true;
};
```



---

#### 选择最长的区块链

区块链中应时刻有且只有一组显式的区块。如果发生冲突（例如，两个节点都生成块号72的区块），我们选择具有最长块数的链。

```js
var replaceChain = (newBlocks) => {
    if (isValidChain(newBlocks) && newBlocks.length > blockchain.length) {
        console.log('Received blockchain is valid. Replacing current blockchain with received blockchain');
        blockchain = newBlocks;
        broadcast(responseLatestMsg());
    } else {
        console.log('Received blockchain invalid');
    }
};
```

---



#### 与其他节点通信

区块链节点的一个重要任务是与其他节点共享和同步区块链。以下规则用于保持网络同步。

- 当一个节点产生一个新块时，要将这个区块广播到网络中。|
- 当一个节点连接到一个新的对等节点时，要查询最新的区块。|
- 当一个节点遇到一个索引大于当前已知块的块时，将该块添加到当前链中，或者查询完整区块链。|

---

![img](https://ask.qcloudimg.com/draft/1046487/mgi4tfvbvc.png?imageView2//0/w/1620)

---

#### 架构

应该注意的是，每个节点实际上公开了两个Web服务器：一个用于控制节点（HTTP服务器），一个用于节点之间的对等通信（Websocket HTTP服务器）![img](https://ask.qcloudimg.com/draft/1046487/sb8oswq2ls.png?imageView2//0/w/1620)

---



![O6C6[NL{P1SHV$XX}]V$AFL](http://oocfz31zv.bkt.clouddn.com/O6C6%5BNL%7BP1SHV%24XX%7D%5DV%24AFL.png)

溯源性：每个区块都会包含上个区块的hash值，可以根据这个Hash值找到上个区块，有点像数据结构中的链表。

所有被连接在一起的区块被称为区块链（Block-Chain）。

Note:![103](http://oocfz31zv.bkt.clouddn.com/103.jpg)

![102](http://oocfz31zv.bkt.clouddn.com/102.jpg)

- `Index (Block #):` 第几个区块? (创世区块链的索引为0)
- `Hash:` 当前区块的hash值
- `Previous Hash:` 上一个区块的hash值
- `Timestamp:`当前区块创建时的时间戳
- `Data:` 存储在当前区块上的交易信息
- `Nonce:` 在找到有效区块之前，我们经历的迭代次数（表示工作量证明演算法进行的次数
- ）

---
### 篡改？
Note:
nonce:Nonce是或Number once的缩写，在密码学中Nonce是一个只被使用一次的任意或非重复的随机数值。nonce 避免重放攻击（Replay attack）

为了确保 nonce 在特定上下文中仅仅被使用一次，可以使用以下策略生成 nonce：
- nonce 可以是一个时间相关变量
- nonce 可以是一个通过足够随机算法生成的足够长的 bits
  当data改变后hash值也跟着改变，但是在只有hash值开头是4个0才是合法的，挖矿的过程实际上就是猜nonce的值，当猜中了Hash的值就会变成0000开头，然后开始广播给所有的节点，节点开始验证合法性，通过后将这个区块添加到区块链的主链之中。
  ![104](http://oocfz31zv.bkt.clouddn.com/104.jpg)

当修改了某条数据之后，之后的每一条数据都行要重新挖矿，如果链很长的话，修改成本非常的高，任何一笔资料被篡改的几率微乎其微。

---

### 少数服从多数

Note:

![107](http://oocfz31zv.bkt.clouddn.com/107.jpg)



每个节点Peer可以包含多个区块(并且区块内容相同)

当某个节点的数据篡改后，即使挖矿后，hash值变成0000开头，但是其他节点的hash值没变，这个被修改过的节点就会被抛弃。

---

## 什么是区块链

- 是一种分布式数据库 |
- 最初是广泛使用在Bitcoin |
- 维护一份连续不断的交易记录档 |
- 每一笔资料被称为一个区块（Block） |
- 每一个区块可以包含一笔以上的交易 |
- 每个区块都会与另一个区块产生连接 |
- 每个区块都会包含上个区块的hash值 |
- 所有被连接在一起的区块被称为链（Chain） |
- 区块链就是由多个区块组成的链 |

---
## 区块链如何工作？

+++

- 产生区块链的过程会经过复杂的密码学运算，复杂的密码学运算可以杜绝记录篡改和修订， |
- 每个区块在成功产生之后是无法修改的。产生区块的过程you称为 挖矿 (Mining)， |
- 负责产生区块的人又称为矿工(Miner),负责产生区块的机器有被称为挖矿机。 |
- 每一笔资粮都可以通过连接找出所有可靠的历史资料 |
- 由于分布式数据库，所以具有去中心化的特性。 |
- 去中心化意味着资料会分布到多个节点上。 |
- 所有节点共同维护整个分布式数据库 |
- 共同维护整份资料库意味着没人可以说自己的区块才是合法有效的区块 |
- 共同维护整份资料库意味着多数人验证过后的区块才是合法有效的区块 |
- 意味超过50%的节点验证通过的区块才是合法有效的区块 |

---


- 每个节点必须存储所有的区块 |

- 每一个节点都可以协助验证区块的有效性 |

- 任何一笔资料被篡改都将破坏区块的完整性 |

- 任何一笔资料被篡改的几率微乎其微 |

- 区块链的运作原理跟Git分布式版本控制系统有着异曲同工之妙 |

- 区块链的包含两种物件类型 交易和区块 |

---


# 情景解说

- 数字货币 

  - 交易物件 就是转账过程的完整内容

    ![105](http://oocfz31zv.bkt.clouddn.com/105.jpg)


  - 区块物件 包含一系列交易的集合

    ![106](http://oocfz31zv.bkt.clouddn.com/106.jpg)

---

  - 区块之间连接起来就是一本账簿 |

  - 区块链基本原理就是让所有人共同维护一个账簿 |

  - 这份共同维护的账簿必需由大伙共同认证 |

---



#以Bitcoin转账交易为例

  ![98](http://oocfz31zv.bkt.clouddn.com/98.jpg)

---



Note:

  - Bitcoin的生态系统中，有着许多运行地电脑<mark>节点</mark>，每个节点就是一台挖坑机，它们专门用来帮大家完成交易，建立新的区块并加入账簿，当A想要转账给B时，要先由B建立一个地址（匿名），把地址交给A之后，再由A建立一个交易物件并通过广播的形式发布到Bitcoin的网络系统中。注意：每个人都可以申请一个地址（一串乱码），该地址可用来付账收款，该地址在变更之后就无法使用（一次性），变更之后就只能用新的地址来交易
  - BItcoin系统会自动挑选不同的节点来验证这笔交易的可靠性，通常一个交易会获得多个不同的节点来进行验证，被选中的节点至少会验证一个以上的区块（即本次交易所需的所有区块）来运算出本次交易产生的合法区块。
  - 基本上每个节点都能产生区块，并且会相互验证彼此之间的有效性。当超过50%的节点说这个节点是合法有效的，这个区块就会被写入账簿，最后再将这些区块通过广播的方式发布回Bitcoin系统中，通过一次交易需要花费十多分钟才能确认，当Bitcoin系统验证交易确实完成。A跟B就可以看到这一笔（合法有效的交易），并且该交易被记录在一个特定的区块之中，并且可以查看自己账户中有中有多少钱。
  - 事实上，并不是A和B拥有这个账簿，而是整个Bitcoin系统中只有一份账簿（所有人的交易记录都在这里），并且分散存储在不同的节点中，每个节点都拥有完整的账簿，因此这本账簿一份完全分布式，去中新化，公开透明，无法篡改，又能匿名交易的一本账簿。

---



## 区块链特点与应用

  - 完全开源的技术

  - 去中心化分布式架构

  - 所有节点都以匿名方式存在，共同验证整份账簿（共同验证交易的节点会通过共识算法决定这是否为合法交易）

  - 任何人 想篡改历史交易记录都将付出极高的代价（你必须让网络上所有包含你这笔交易的区块同时修改方位有效）

  - 完全自动化的冲突处理方式（比如：同一笔交易在不同节点同时完成的情况）

---

### 应用领域

    - 数字货币
    - 能将数据去中心化，透明化，不可篡改仅能新增区块，永久保存的应用都适用于区块链技术
      - 数字资产管理
      - 供应链透明化
      - 生产履历，产销履历
      - 智能合约，契约执行
      - 交易流程透明化
      - 电子函证



---

## 比特币调节难度和总数分配固定

![113](http://oocfz31zv.bkt.clouddn.com/113.jpg)

Note:
最初的奖励为每个区块50BTC，每2.1万个区块后减半，所以通胀速度会几何级数下降，无限逼近2100万的总量上限。容易看出，这是一个公比为0.5的等比数列，而这个数列的和为210000×50（1+0.5+0.25+0.125+……）=2100万，即BTC总量2100万的来源。在比特币系统里面，打包一个区块的时间大约是十分钟。也就是说理论上，每210万分钟，新币的发行速度就会发生一次减半，换算成年就是大约3.995年。

这就是江湖上流传的“四年减半”这个说法的由来。由于比特币的算力一直在快速增加，导致系统平均出块时间略小于10分钟，因此实际上的减半时间比理论值3.995还要更短一些。

---

比特币挖矿本质上是计算一个hash值，最后得到的结果小于一个大家公认的数，大家就承认你挖到了。举个最简化的模拟（实际算法和细节都不一样的）:

sha256(timt.time()*m ) < difficult

指定difficult，求m，谁先找到m值谁就求解成功。

很明显，difficult越小，这个m值越难求解，我们把difficult称之为难度。

---



求解m的人会越来越多，本来平均10分钟会有人求解成功，变成平均5分钟就有人算出来了，而每次求解成功，系统都会发放奖励，这样就会违背比特币系统生产速率恒定的初衷，于是系统会自动减小difficult，也就是所谓的提升难度。提升周期大概为2周一次。

难度并不总是上升的，如果算力下降，难度也会下降，这样就维持比特币产出在长时间周期内大概是平稳的。

后辍是DAT的文件，均为数据库文件。其中最重要的是钱包密匙文件Wallet.dat，要保管好，不要误删除，不被别人复制。可以改名另存，当然，改了名，程序文件就认不得它了，又会生成一个新的Wallet.dat，里边的地址是不相同的，要分清此Wallet=1#.dat和彼Wallet=2#.dat，要用回原来地址，就将相应的文件改回原名来用。

总结一下，一个虚拟币用户端，实际就是两大文件夹内容。一个夹是下载安装得到的程序文件，另一个夹是按设置运行已安装好的程序所得到的并随使用过程增长的数据文件。

更深层次的诠释什么是“钱包”，“钱包”这个概念，因人而异有较多的不同定义。我们在这里说的是分布式加密传输用门户终端记录数据库。钱包就是一个互联网上分布在各处用户的数据库。基本构成是程序文件夹内容和数据夹内容。其中，有个人权益资料和公共记录资料。Wallet.dat文件是重要的个人权益资料，它用在公共资料数据库平台上获得权益归属和处置权利。

---

## 什么是merkle tree

假设你已经知道了什么是哈希算法以及哈希是用来干啥的。

网络传输数据的时候，A收到B的传过来的文件，需要确认收到的文件有没有损坏。如何解决？

有一种方法是B在传文件之前先把文件的hash结果给A，A收到文件再计算一次哈希然后和收到的哈希比较就知道文件有无损坏。

但是当文件很大的时候，往往需要把文件拆分很多的数据块各自传输，这个时候就需要知道每个数据块的哈希值。怎么办呢？

----

这种情况，可以在下载数据之前先下载一份哈希列表(hash list)，这个列表每一项对应一个数据块的哈希值。对这个hash list拼接后可以计算一个根hash。实际应用中，我们只要确保从一个可信的渠道获取正确的根hash，就可以确保下载正确的文件。

**似乎很完美了。但是还不够好！**

上面基于hash list的方案这样一个问题：

---



**有些时候我们获取(遍历)所有数据块的hash list代价比较大，只能获取部分节点的哈希。**

有没有一种方法可以通过部分hash就能校验整个文件的完整性呢？

答案是肯定的，merkle tree能做到。它长这样子:

---![这里写图片描述](http://static.open-open.com/lib/uploadImg/20140403/20140403101532_513.jpg)

在上图中，叶子节点node7的value（v7) = hash(f1),是f1文件的HASH;而其父亲节点node3的value = hash(v7, v8)，也就是其子节点node7 node8的值得HASH

---

特点如下:

1、数据结构是一个树，可以是二叉树，也可以是多叉树（本BLOG以二叉树来分析）

2、Merkle Tree的叶子节点的value是数据集合的单元数据或者单元数据HASH。

3、Merke Tree非叶子节点value是其所有子节点value的HASH值。



---

很明显，这种结构跟hash list相比较，根哈希不是用所有的数据块哈希拼接起来merkle tree应用在区块链上

下面是本文的重点。

比特币系统的区块链中，每个区块都有一个merkle tree

可以看到merkle root哈希值存在每一个区块的头部，通过这个root值连接着区块体，而区块体内就是包含着大量的交易。每个交易就相当于前面提到的数据块，交易本身有都有自己的哈希值来唯一标识自己。计算的，而是通过一个层级的关系计算出来的。

****