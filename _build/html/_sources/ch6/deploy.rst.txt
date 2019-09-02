部署智能合约
===============

智能合约的部署过程也是一次普通的交易过程，我们需要将智能合约的数据整合到交易体的数据区（data），并发送出去。 一旦交易被捕获且挖矿完成，我们的合约就已经部署在了区块链上并具有了一个独一无二的地址。通过该地址我们就能和智能合约通信并调用合约的方法

承接上文，我们整理部署交易所需要的数据：ABI 和 bin 字节码。

.. code-block:: bash
   :linenos:

   > var myAbi = output.contracts['Vault.sol:Vault'].abi
   undefined
   > var myContract = eth.contract(JSON.parse(myAbi))
   undefined
   > var myBinCode = "0x" + output.contracts['Vault.sol:Vault'].bin
   undefined

我们整理了3个变量 ``myAbi`` 、``myContract`` 与 ``myBinCode`` 。我们已经准备好了部署所需的所有变量，下面开始智能合约的部署。

首先解锁持有以太币的账户。

.. code-block:: bash
   :linenos:

   > personal.unlockAccount(eth.accounts[0], '123', 300)
   true

接着调用特殊的合约生成方法（其底层也是一个交易发送方法）来部署合约。

.. code-block:: bash
   :linenos:

   > var deployObject = { from: eth.accounts[0], data: myBinCode, gas: 1000000 }
   undefined
   > var myContractInstance = myContract.new(deployObject)
   undefined

此时挖矿的 Geth 节点窗口将持续打包挖矿，并出现一条接收到合约创建指令的日志：

.. code-block:: bash
   :linenos:

   INFO [09-16|23:04:23.160] Submitted contract creation   fullhash=0xbd0d5a8b1b24500e7d9ee49d7170aaa24f017f402b0964bcafed717ea7c8e8ca contract=0x1f0723b71f5824567E9aCc1f1079E91FCd958a50

挖矿成功！合约已经部署到了区块链上，并具有了自己的独一无二地址。

.. centered:: 0x1f0723b71f5824567E9aCc1f1079E91FCd958a50

这个地址是根据发送者的地址0x53dc408a8fa060fd3b72b30ca312f4b3f3232f4f 与发送者的发送交易数量 nonce 唯一计算的出的地址。我们可以通过创建的变量myContractInstance 来查看我们和合约的详细情况。

.. code-block:: bash
   :linenos:

   > myContractInstance
   {
     abi: [{
         constant: false,
         inputs: [{...}],
         name: "set",
         outputs: [],
         payable: false,
         stateMutability: "nonpayable",
         type: "function"
     }, {
         constant: true,
         inputs: [],
         name: "get",
         outputs: [{...}],
         payable: false,
         stateMutability: "view",
         type: "function"
     }],
     address: "0x1f0723b71f5824567e9acc1f1079e91fcd958a50",
     transactionHash: "0xbd0d5a8b1b24500e7d9ee49d7170aaa24f017f402b0964bcafed717ea7c8e8ca",
     allEvents: function(),
     get: function(),
     set: function()
   }

