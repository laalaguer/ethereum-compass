:github_url: https://github.com/laalaguer/ethereum-compass/blob/master/ch10/shortattack.rst

短地址攻击
=================

本小节需要读者熟读本书中的第9章ERC20资产合约以及第7章虚拟机的原理。尚未阅读的读者可以在上述章节阅读完毕后再来阅读本章节。

所谓“虚拟机的安全”问题，我们在之前章节的ERC20资产合约中已经提到采用SafeMath库来校准数据在程序中的溢出问题。除了这个编程问题以外，更有一个很大的问题存在于虚拟机对“输入数据的补足”上。Golem团队2017年4月份提出了一个称之为短地址攻击的以太坊攻击手法。透过这个攻击原理我们可以看到以太坊虚拟机在内部数据检查上的不足。

我们先来回顾一下ERC20数字资产合约的transfer()函数，以及该函数的调用。

.. code-block:: solidity

   /** Make a Transfer.
    * @dev This operation will deduct the msg.sender's balance.
    * @param _to address The address the funds go to.
    * @param _value uint256 The amount of funds.
   */
   function transfer(address _to, uint256 _value) public returns (bool) {
     require(_to != address(0), "Cannot send to all zero address.");
     require(_value <= balances[msg.sender], "msg.sender balance is not enough.");
   
     // SafeMath.sub will throw if there is not enough balance.
     balances[msg.sender] = balances[msg.sender].sub(_value);
     balances[_to] = balances[_to].add(_value);
     emit Transfer(msg.sender, _to, _value);
     return true;
   }

这个函数的内部非常“漂亮”，先是检查了地址是否为全0；再检查了调用方（也就是发送方）的数字资产余额是否够本次传输；最后调用SafeMath函数安全地执行账户余额增减操作，并记录一个日志Transfer事件。那么，这个函数究竟在调用时有何问题呢？

问题就出在具体调用的函数上我们传入的数据。按照我们所学，智能合约的调用采用“确定合约地址 ->确定合约ABI ->两者结合，新建可操纵实例对象 ->调用该实例对象的transfer()方法 ->结束”的步骤实施。那么好，假设我们已经有如下的ERC20合约。
  - 合约地址：0xa94a33f776073423e163088a5078feac31373990
  - transfer调用方：0xd5734756ba72dd46c3e4c87fbfa7acd1c1310e14
  - transfer受益方：0x430D3F5a337Ea03e6979c6Dce2850F080F901Ab7
  - transfer 数量：2,000,000,000 

那么调用transfer函数的时候(JavaScript为例)就是如下形式。

.. code-block:: javascript 

   erc20_instance.methods
     .transfer("0x430D3F5a337Ea03e6979c6Dce2850F080F901Ab7", "2000000000")
     .send({from: deployer_address}, function(error, txHash){
       if (error){
         console.log('tx error:', error)
       }
       if (txHash){
         console.log('tx hash:', txHash)
       }
     })

那么EVM看到的形式时什么呢？是如图的形式。

.. _10-1:
.. figure:: /img/Picture55.png
   :align: center
   :width: 600 px

   EVM眼里的交易形式


这数据我们一眼看过去就能辨别出几个部分。

  - transfer 的函数签名：a9059cbb000000000000000000000000
  - 收益方：430d3f5a337ea03e6979c6dce2850f080f901ab7
  - 转移数量Hex表达：6765c793fa10079d0000000

这点可以用node命令行来验证。

.. code-block:: bash

   > parseInt('0x6765c793fa10079d0000000')
   2e+27

问题其实出在下图所示的Data中间那些个0上，从地址 ``ab7`` 结尾到数量 ``676`` 开头之间多了好多0。

.. _10-2:
.. figure:: /img/Picture56.png
   :align: center
   :width: 600 px

   交易出漏洞的部分

根据EVM解析器，这个调用的规定其实是如下的（客户端来满足）。
  - 开头4字节是函数签名。
  - 中间32字节是address _to（转账的目标地址），不足，则高位补0。
  - 末尾32字节是uint256 _value（转账金额），不足，则高位补0。

哦不！这里有一个很大的问题，就是收益方_address_to和转账金额 _value都是我们用户手动填入的。如果我们的转账地址恰好是结尾有0的，如下地址。

.. centered:: 0x1111111111111111111111111111111111111100

这个地址结尾处有0环绕，我们先看一下正常发起的一笔交易（136个字符）。

a9059cbb0000000000000000000000001111111111111111111111111111111111111100000000000000000000000000000000000000000006765c793fa10079d0000000

当我们组同样一个交易，并且“有意”忽略了地址结尾的2个0，那么会是什么样子呢？它会变成（134个字符）。
a9059cbb00000000000000000000000011111111111111111111111111111111111111000000000000000000000000000000000000000006765c793fa10079d0000000

这时候EVM就会犯错了，它在阅读到函数签名（4字节）之后，紧跟着阅读目标地址（32字节），它运行到了字符串结尾，试图读取转账金额的时候发生了困难，因为最后剩余部分不足32字节了！怎么办？它默认行为是，在结尾补充0，补满32字节！那么我们的转账金额向“左边”顺移了两位。
  - 原转账金额：
  - 00000000000000000000000000000000000000000006765c793fa10079d0000000
  - 现转账金额：
  - 000000000000000000000000000000000000000006765c793fa10079d000000000

这可是一个惊天地的错误。因为这个值是Hex值，也就是16进制表示的，每移动1位，就会乘以16倍，移动两位16x16=256倍。转账金额直接多了200多倍！这个攻击的得名也因特殊的地址截断方式，而称为“短地址攻击”。那么，如何防护这样的攻击呢？

由于这数据最终进入的是EVM虚拟机，所以在到达虚拟机之前，我们可以做如下防护：
  - 智能合约在transfer()函数内强制检查len(msg.data == 68) ，不让输入数据过短。
  - 以太坊节点负责检查接到的交易请求msg.data长度是否合规。
  - 交易所节点负责检查接到的提币请求，地址格式长度是否合规。

以上解决方案还是“挽救”的措施，类似的问题暴露出了以太坊在安排函数地址和函数数据输入的时候，没有使用分隔符号，且虚拟机会自动补全规则的漏洞，最终解决方案还是要靠EVM自身核心代码的改进升级才行。
