:github_url: https://github.com/laalaguer/ethereum-compass/blob/master/ch6/compile.rst

Solc编译智能合约
=======================


我们的简单智能合约已经准备好了，如下所示：

.. code-block:: solidity
   :linenos:
   :caption: vault.sol

   pragma solidity ^0.4.24;

   contract Vault {
       uint vaultData;
       function set(uint data) public{
           vaultData = data;
       }
   
       function get() public view returns (uint) {
           return vaultData;
       }
   }

简单解释一下各个部分。智能合约的名字是Vault，是一个存储合约，它开辟一个存储区 vaultData，该存储区是一个 ``uint`` 类型的变量(unsigned int,正整数)。智能合约一共包含两个方法：set 与get。分别为设置 vaultData 的值和读取 vaultData 的值。整个合约不会产生事件，所以也不会产生日志。合约指定需要编译器版本 ``0.4.24`` 来进行编译


我们来到控制台，执行以下命令编译该智能合约：

.. code-block:: bash

   solc --optimize --combined-json abi,bin,interface Vault.sol


我们在命令中指定 solc 在编译时尽量优化代码，并将编译好的字节码 bin，与合约接口描述 ABI(Applicatoin Binary Interface) [#]_ 组合在一起输出。控制台中立即返回了编译结果。

.. code-block:: javascript

   {"contracts":{"Vault.sol:Vault":{"abi":"[{\"constant\":false,\"inputs\":[{\"name\":\"data\",\"type\":\"uint256\"}],\"name\":\"set\",\"outputs\":[],\"payable\":false,\"stateMutability\":\"nonpayable\",\"type\":\"function\"},{\"constant\":true,\"inputs\":[],\"name\":\"get\",\"outputs\":[{\"name\":\"\",\"type\":\"uint256\"}],\"payable\":false,\"stateMutability\":\"view\",\"type\":\"function\"}]","bin":"608060405234801561001057600080fd5b5060bf8061001f6000396000f30060806040526004361060485763ffffffff7c010000000000000000000000000000000000000000000000000000000060003504166360fe47b18114604d5780636d4ce63c146064575b600080fd5b348015605857600080fd5b5060626004356088565b005b348015606f57600080fd5b506076608d565b60408051918252519081900360200190f35b600055565b600054905600a165627a7a723058203269ba0a634bf05e2a15966872aaa719b6d147aaa419d656374ad860104e6ef40029"}},"version":"0.4.24+commit.e67f0147.Darwin.appleclang"}

我们将该编译结果放入一份 **temp.js** 文件中并重新排版。

.. code-block:: javascript

   var output = {"contracts":{"Vault.sol:Vault":{"abi":"[{\"constant\":false,\"inputs\":[{\"name\":\"data\",\"type\":\"uint256\"}],\"name\":\"set\",\"outputs\":[],\"payable\":false,\"stateMutability\":\"nonpayable\",\"type\":\"function\"},{\"constant\":true,\"inputs\":[],\"name\":\"get\",\"outputs\":[{\"name\":\"\",\"type\":\"uint256\"}],\"payable\":false,\"stateMutability\":\"view\",\"type\":\"function\"}]","bin":"608060405234801561001057600080fd5b5060bf8061001f6000396000f30060806040526004361060485763ffffffff7c010000000000000000000000000000000000000000000000000000000060003504166360fe47b18114604d5780636d4ce63c146064575b600080fd5b348015605857600080fd5b5060626004356088565b005b348015606f57600080fd5b506076608d565b60408051918252519081900360200190f35b600055565b600054905600a165627a7a723058203269ba0a634bf05e2a15966872aaa719b6d147aaa419d656374ad860104e6ef40029"}},"version":"0.4.24+commit.e67f0147.Darwin.appleclang"}

输出结果一共分为两个组成大部分，第一部分是智能合约抽象描述Application Binary Interface：ABI ，它展开后如代码所示。

.. code-block:: javascript
   :linenos:

   [{
     "constant": false,
     "inputs": [{
         "name": "data",
         "type": "uint256"
     }],
     "name": "set",
     "outputs": [],
     "payable": false,
     "stateMutability": "nonpayable",
     "type": "function"
   }, {
     "constant": true,
     "inputs": [],
     "name": "get",
     "outputs": [{
         "name": "",
         "type": "uint256"
     }],
     "payable": false,
     "stateMutability": "view",
     "type": "function"
   }]

ABI 代表了一份“说明书”，展示了对于我们生成的合约究竟可以进行哪些操作、操作的参数又是哪些。
在编译期我们就已经确定了参数类型和函数名称，不存在动态类型。我们研读这份ABI 的总体结构，是一个列表。
列表里共2个项目：名为 ``get`` 的方法与 名为 ``set`` 的方法，其中 set 方法接受一个 ``uinit256`` 的参数，
名为 data（ ``uint256`` 即256位的 ``uint``，在solidity中与 uint 同义）；
get 方法不接受 任何的参数，但是输出一个结果为 uint256 的值，且返回值不命名。
下面对ABI 中常见的几个关键字进行解释。

+-----------------+------------------------------------------------------------+
| 名称            | 解释                                                       |
+-----------------+------------------------------------------------------------+
| type            | 接口类型，默认为function，也可以是construnctor、fallback等 |
+-----------------+------------------------------------------------------------+
| name            | 方法名字                                                   |
+-----------------+------------------------------------------------------------+
| inputs          | 接口输入参数列表，每一项都是参数名+参数类型                |
+-----------------+------------------------------------------------------------+
| outputs         | 接口输出结果列表，每一项都是返回值名+返回值类型            |
+-----------------+------------------------------------------------------------+
| constant        | 布尔值，若为true 则该接口不修改合约存储区，是只读方法      |
+-----------------+------------------------------------------------------------+
| payable         | 布尔值，标明该方法是否接受以太币                           |
+-----------------+------------------------------------------------------------+
| stateMutability | 枚举类型，为下列选项之一：                                 |
+-----------------+------------------------------------------------------------+
|                 | pure：表明该方法只读不修改存储，且不读取区块链状态         |
+-----------------+------------------------------------------------------------+
|                 | view：表明该方法只读不修改存储，但读取区块链状态           |
+-----------------+------------------------------------------------------------+
|                 | nonpayable：该方法不能接受以太币                           |
+-----------------+------------------------------------------------------------+
|                 | payable：该方法可以接受以太币                              |
+-----------------+------------------------------------------------------------+

第二部分是 bin 也就是经过编译器优化过后的可运行的合约字节码。真正部署到区块链上的是 bin 这一部分的代码，这部分代码的内容是初始化代码，包含了如何清理空间、创建变量、初始化合约的指令。


------------------------

.. [#] 笔者注：ABI定义见 https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI