:github_url: https://github.com/laalaguer/ethereum-compass/blob/master/ch6/prepare.rst

智能合约发布准备
============================

我们将这份智能合约部署到Geth节点运行的私有区块链上，如果还未开始，请按如下命令启动Geth节点并启用挖矿。

.. code-block:: bash
   :linenos:

   cd ether-test
   geth --datadir ./db/ --rpc --rpcaddr=127.0.0.1 --rpcport 8545 --rpccorsdomain "*" \
      --rpcapi "eth,net,web3,personal,admin,shh,txpool,debug,miner" \
      --nodiscover --maxpeers 30 --networkid 198989 --port 30303 \
      --mine --minerthreads 1 \
      --etherbase "0x53dc408a8fa060fd3b72b30ca312f4b3f3232f4f"

好，Geth 节点开始稳步挖矿，区块链已经运行。我们再用attach命令依附到正在运转的Geth节点上，获得操作台的控制权。

.. code-block:: bash
   :linenos:

   geth --datadir ./db attach ipc:./db/geth.ipc
   
   Welcome to the Geth JavaScript console!
   
   instance: Geth/v1.8.14-stable/darwin-amd64/go1.10.3
   coinbase: 0x7df9a875a174b3bc565e6424a0050ebc1b2d1d82
   at block: 90 (Sun, 16 Sep 2018 15:56:37 CST)
    datadir: /ether-test/db
    modules: admin:1.0 debug:1.0 eth:1.0 ethash:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0 web3:1.0

我们导入刚才建立好的 temp.js 文件。

.. code-block:: bash
   :linenos:

   > loadScript('/path/to/your/ether-test/temp.js')
   true
   > output
   {
     contracts: {
       Vault.sol:Vault: {
         abi: "[{\"constant\":false,\"inputs\":[{\"name\":\"data\",\"type\":\"uint256\"}],\"name\":\"set\",\"outputs\":[],\"payable\":false,\"stateMutability\":\"nonpayable\",\"type\":\"function\"},{\"constant\":true,\"inputs\":[],\"name\":\"get\",\"outputs\":[{\"name\":\"\",\"type\":\"uint256\"}],\"payable\":false,\"stateMutability\":\"view\",\"type\":\"function\"}]",
         bin: "608060405234801561001057600080fd5b5060bf8061001f6000396000f30060806040526004361060485763ffffffff7c010000000000000000000000000000000000000000000000000000000060003504166360fe47b18114604d5780636d4ce63c146064575b600080fd5b348015605857600080fd5b5060626004356088565b005b348015606f57600080fd5b506076608d565b60408051918252519081900360200190f35b600055565b600054905600a165627a7a723058203269ba0a634bf05e2a15966872aaa719b6d147aaa419d656374ad860104e6ef40029"
       }
     },
     version: "0.4.24+commit.e67f0147.Darwin.appleclang"
   }

至此我们已经将编译好的智能合约顺利导入了开发环境中。
