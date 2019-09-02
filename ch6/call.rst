:github_url: https://github.com/laalaguer/ethereum-compass/blob/master/ch6/call.rst

调用智能合约
======================

既然我们创建，并援引了合约的实例，接下来围绕着该合约实例我们就可以发送交易（消息）来通讯并且设置合约中的存储值了。下面我们来获取Vault合约中的vault存储值。由于get方法并不会修改区块链上的存储区，所以呼叫这个方法是在Geth节点上通过查询立即实施的，并不会产生交易。

.. code-block:: bash
   :linenos:

   > myContractInstance.get.call()
   0  

此时vault值为 ``0`` 。我们来通过发送交易修改该值。我们调用合约的set方法。

.. code-block:: bash
   :linenos:

   > myContractInstance.set.sendTransaction(17, { from: eth.accounts[0], gas: 1000000})
   "0x983d92b8ca0d3fc7b2e26ed3c0b6a93221adaa032dd6fe02a5b00378b190a898"

交易签名成功！函数调用返回了我们一个交易的哈希值，该交易已经发出。

当矿工成功接到交易后，会显示如下的日志，打包成功后，存储区被改变了。

.. code-block:: bash

   INFO [09-16|23:19:01.430] Submitted transaction   fullhash=0x983d92b8ca0d3fc7b2e26ed3c0b6a93221adaa032dd6fe02a5b00378b190a898 recipient=0x1f0723b71f5824567E9aCc1f1079E91FCd958a50  

我们再来调用一次get方法来验证一下存储区。

.. code-block:: bash
   :linenos:
   
   > myContractInstance.get.call()
   17

至此，我们已经对一个简单的智能合约进行了部署和调用。麻雀虽小、五脏俱全，任何复杂的智能合约在读取和写入操作时候不外乎也是由工具在幕后执行了上述的操作。读者可以领会到 bin和 ABI 两者在部署和调用过程中分别起到了实体和“说明书”的角色，bin 是实际运行在以太坊虚拟机中的二进制合约代码，而 ABI则供调用者按图索骥去寻找合约方法的参数和位置，方便使用者进行准确的调用。
