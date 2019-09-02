:github_url: https://github.com/laalaguer/ethereum-compass/blob/master/ch2/account.rst

.. _reference-account:

账户探秘
====================

探索以太坊世界的第一步，是创建一个账户，就如同在银行开户一样。
账户是安全地进行以太坊交易的基础。
与在银行开户不同的是，以太坊的账户可以离线生成而不需要得到任何工作人员的许可，
并且这些账户是 **完全匿名** 的。
适用于生成账户的开源工具有很多，如网页工具、桌面软件、手机APP等。
它们遵循同一套账户生成标准。
一个用户也可以同时生成、保存、持有多个账户。
下面我们讲述账户的原理与账户生成的过程，以及账户状态与全球状态的关系。

账户与账户状态
-----------------------
以太坊的账户共分成两类

:guilabel:`外部账户` (Externally Owned Account, **EOA** ) 与 :guilabel:`智能合约` (Contract Account, **CA** )。 

外部账户由 **一把私钥** 与该私钥对应的公开地址来表示。在一般情况下，私钥掌握在用户的手中。

智能合约账户 **没有私钥** ，仅有公开的地址，它的行为由合约自身包含的代码逻辑来控制。

账户的状态(Acccount State)描述了一个账户当前的情况。
以太坊公链时时刻刻跟踪并维护着每一个账户的状态。
一个账户在初次接收或者发出交易后，都会形成初始状态。
随着时间的推移，每次针对该账户的交易将不断修改其状态。

总结而言，每一个账户在数据结构上具有两个元素：一个公开地址，一个与该地址关联的状态，如下图所示。

.. figure:: /img/Picture8.png
   :align: center
   :width: 600 px

   一个账户



账户状态的内涵
----------------------------

那么，具体的账户状态包含一些什么呢？
账户状态包含四大元素：

   - 已执行交易总数 :guilabel:`nonce`，用来标示该账户发出的交易数量；
   - 持币数量 :guilabel:`balance`，记录用户的以太币余额；
   - 存储区的哈希值 :guilabel:`storage hash` ，指向智能合约账户的存储数据区；
   - 代码区的哈希值 :guilabel:`code hash`，指向智能合约账户存储的智能合约代码。

下图显示了外部账户与智能合约的账户状态。

.. figure:: /img/Picture9.png
   :align: center
   :width: 600 px

   外部账户与智能合约账户的结构对比


接下来，对以上各个名词进行详细解释。

已执行交易总数
^^^^^^^^^^^^^^^^^^^^

该值会随着用户不断发送交易而递增，保障用户发出的交易是按照顺序被收纳入最终的区块链。
因为在同一个账户中，已执行交易总数不可以在区块链中再次出现。
当用户创建智能合约时，要指定合约地址，该地址是由用户账户的已执行交易总数和用户账户地址联合计算而得出的。

.. Note::
   假设我拥有一个账户，该数值为 ``13`` 如果我给人转账，则该数字增加到 ``14``

持币数量
^^^^^^^^^^^^^^^

持币数量包含了该账户当下可花费的以太币的数量。外部账户和智能合约都可以持有以太币。

.. Note::
   指定了可以接收以太币的智能合约也可以像自然人一样，持有以太币！

存储区的哈希值
^^^^^^^^^^^^^^^^^^^^^

该值为 **智能合约独有** ，外部账户不包含该值。
存储区即为智能合约在运行中，产生的数据的存储地。
在合约的生命周期里，该区域的内容被合约代码不断写入、读取。
存储区存放于以太坊网络节点的硬盘上。
存储区的内容通过散列函数得出校验哈希值，该值即为存储区的哈希值。

.. Note::
   存储区相当于智能合约的“小硬盘”。

代码区的哈希值
^^^^^^^^^^^^^^^^^^^^^^^

该值为 **智能合约独有**，外部账户不包含该值。
代码区即为智能合约代码本身。
在合约的生命周期中，该区域的内容是不可更改的 **只读状态**。
代码区存放于以太坊网络节点的硬盘中，当运行时被读入虚拟机执行。代码区的内容通过散列函数得出校验哈希值，该值即为代码区的哈希值。

.. Note::
   代码区相当于智能合约的“程序”部分。

.. Note::
   哈希算法就是通过一定的数学算法 y=Ϝ(x) 的单向函数，将不定长的输入值，
   经过函数变换后变成定长的哈希值。
   这个数学算法是不可逆向运算操作的（意即不可通过输出推断输入，却可通过输入轻松运算出输出），
   并具有良好的抗碰撞特性。
   唯一的输入对应了唯一输出，哪怕是改动一个输入字符，都可以让输出哈希值产生翻天覆地的变化。[#]_
   在数据校验领域中，哈希算法被用来对文档进行签名，以防止文档中途被篡改或者丢失字符。
   在区块链中常用的安全哈希算法是 SHA3-256算法，即输出定长为256位的第三代哈希算法。[#]_

没有钱包App, 如何生成账户？
---------------------------------

普通用户最频繁使用的账户主要是外部账户(Externally Owned Account, EOA)。
这个账户可以用来发送/接受以太币，也可以发起部署智能合约的行为。
以太坊的外部账户仅由私钥(private key)与它所相对应的公开地址(address)组成。

**Okay, 第一步，我如何生成私钥？**

:guilabel:`私钥` 是一个32 bytes (256 bits) 长度的随机数。用户需要一个可靠的随机源来产生该随机数，该随机数取值在0~2 :sup:`256` 之间。
私钥的举例如下所示(16进制表示)。生成私钥的逻辑如代码清单2-1所示。

+------+------------------------------------------------------------------+
| 私钥 | bdb2c8d55b47e7c37dabdead589eec3d463b2de656ed6ba9b75143e72180ae09 |
+------+------------------------------------------------------------------+

.. code-block:: javascript
   :caption: 代码清单2-1

   const randomBytes = require('randombytes')
   /**
   * Create a random private key buffer.
   * @returns {Buffer} private key: a 32 bytes buffer
   */
   const createRandomPrivateKey = function (){
     const privateKey = randomBytes(32)
     return privateKey
   }

在代码清单2-1中主要逻辑是生成一个32字节长度的随机数。我们选用了 Javascript 的 ``randombytes`` 库函数辅助我们生成该随机数。

.. WARNING::
   当选择生成私钥的随机数方法时，需要选择满足密码学强度的随机数生成方法，计算机软件本身是无法生成真正随机数的，在长周期的情况下必然会出现相同的随机数。操作系统通过维护一个熵池收集来自设备的噪音: 鼠标移动、键盘按键等等。熵值越大，代表系统无序性越大，利用熵生成的随机数也越不可捉摸。当使用其他语言编程时，请选用相应可靠的随机数发生器


**第二步，公开地址是如何从私钥派生的呢？**

这分为几个步骤：首先，我们特殊选定的椭圆曲线(ECDSA-secp256k1)算法 [#]_，
代入 :guilabel:`私钥` 作为参数进行运算，得出的结果为 :guilabel:`公钥`。

这个过程是不可逆的，并且是唯一与私钥对应的。
其次，在生成公钥后，再将其进一步放入一个哈希算法生成哈希值，截取哈希值的最后40位16进制字符得到地址(160 bits或20 bytes)。
对于上述我们举例的的私钥，由其派生的 :guilabel:`地址` 如下表所示。

+------+------------------------------------------------------------------+
| 地址 | 0xda36cd6F5aF1CA5A226c02B3BD74E3F1BA354B9F                       |
+------+------------------------------------------------------------------+
| 私钥 | bdb2c8d55b47e7c37dabdead589eec3d463b2de656ed6ba9b75143e72180ae09 |
+------+------------------------------------------------------------------+

有了地址，你朋友就可以给你打以太币了！生成该地址的代码如代码清单2-2所示。

.. code-block:: javascript
   :caption: 代码清单2-2

   const secp256k1 = require('secp256k1')
   const keccak = require('keccak')
   /**
    * Turn private key into address
    * @param privateKey {Buffer} 32 bytes of private key
    * @returns {Buffer} 20 bytes of address
    */
   const privateKeyToAddress = function (privateKey) {
       // 32 bytes of private key buffer to generate 65 bytes of public key.
       // Get rid of 0x04 at the begin of public key. (65-1=64 bytes remains)
       const publicKey = secp256k1.publicKeyCreate(privateKey, false).slice(1)
       // Take right-most 20 bytes and turn to hex representation.
       return keccak('keccak256').update(publicKey).digest().slice(-20)
   }


上述代码的执行逻辑解释如下：

   - ``32`` 字节私钥生成的长度为 ``65`` 字节的公钥。
   - 删除为首的一个字节 ``0x04`` ，还剩 ``64`` 字节。
   - 将其放入 ``keccak256`` 哈希算法，生成一个 ``256`` 位的哈希值。
   - 截取哈希值的最后 ``20`` 字节， 即为所求的公开地址。
   - (可选）辅以 ``0x`` 的开头装饰，表明这是一个16进制的书面记录形式。

**我生成的账户安全吗？**

和一般的网站申请账户不同， **加密货币的账户仅需要可靠的软件在离线状态下生成** ，
而不需要去特殊网站进行注册。
很多虚拟货币交易所的管理大额虚拟货币的账户都是通过上述方法在一台离线的计算机上生成的。
那么，如何保证每次生成的私钥不是已经被他人生成过的？在现实中，两个私钥碰撞的概率有多大呢？

我们已知：私钥地址空间有 2 :sup:`256`，而宇宙中的已知原子总数有 10 :sup:`80`, 两者比较谁大谁小？我们做一个除法。

:math:`2^{256}  ÷ 10^{80} = 1.1579209e+69 = 10^{69}`

从上述算式可以看出，私钥空间比我们宇宙空间的原子总数的倍数还要多。
可以说在全人类都参与使用加密货币的情况下，即使每次交易都使用新的地址， 碰巧遇上他人私钥的概率比生活中选中一个原子去砸中另外一个原子的概率还要小。

你生成的账户，是安全的。

.. Note::
   生成账户的具体Javascript代码可以参考笔者的Github项目：http://github.com/laalaguer/VeChain-Address/


智能合约地址的生成
-------------------------

看到这里，你会问，好的，我已经清楚如何生成我的钱包了。但是智能合约也有账户，它是如何生成的呢？

与外部账户不同，智能合约账户的地址创建并非由外部促成，而是在创建合约时候由代码自动生成的。智能合约账户有公开的地址，却没有对应的私钥，这意味着：

   - 合约转出以太币，并非通过私钥签名方式。
   - 只有合约自身的逻辑代码能够管理它的以太币，除极少数例外（例如合约创建者销毁合约，合约收到的以太币将默认打给该创建者）。

我们将在动手实践环节中，用 ``web3`` 向读者展示合约的部署生成过程，在这里仅演示当创建一个合约时，究竟发生了什么。代码如清单2-3所示。

.. code-block:: javascript
   :caption: 代码清单2-3

   const rlp = require('rlp');
   const keccak = require('keccak');

   var nonce = 0x00; // Nonce of sender.
   var sender = '0x6ac7ea33f8831ea9dcc53393aaa88b25a785dbf0'; // Sender address.

   var input_arr = [sender, nonce];
   var rlp_encoded = rlp.encode(input_arr);

   var contract_address_long = keccak('keccak256').update(rlp_encoded).digest('hex');
   var contract_address = contract_address_long.substring(24); //Trim.
   console.log("contract_address: " + contract_address);

上述生成地址的过程依赖于两个关键参数：

   - 合约创建人（发送方）的账户地址(``20字节``)。
   - 发送方账户内的已发生交易总数 nonce 值。

为了得出合约的部署地址，将上述两个参数放入 :guilabel:`RLP` 函数进行编码，经由 keccak256 哈希算法算出哈希值，最终取出结果中的20 bytes，将其设定为合约地址。

:guilabel:`RLP` (Recursive Length Prefix)函数 [#]_ ，全名递归长度前缀编码函数，是以太坊序列化所采用的序列化和反序列化的主要方式。在进行网络传输、数据库存储之前，二进制数组数据都会经过这个函数进行编码，该函数的详细定义请参见以太坊的官方维基 以及本书4.2章节。


.. [#] Hash Function (2019),Wikipedia, Available at: https://en.wikipedia.org/wiki/Hash_function
.. [#] Guido Bertoni, et al, ‘SHA3’ (2015),Wikipedia, Available at: https://en.wikipedia.org/wiki/SHA-3
.. [#] Darrel Hankerson, et al (2004) ‘Guide to Elliptic Curve Cryptography’, Springer.
.. [#] Ethereum Community Authors (2019), ‘Recursive Length Prefix RLP’, The Ethereum Wiki, Available at: https://github.com/ethereum/wiki/wiki/RLP
