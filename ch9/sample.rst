Truffle启动样例项目
==========================

Truffle很友好，当读者第一次运行Truffle的时候，可以什么参数也不指定，创建一个空白项目，或者 Truffle 代为下载一个模板示例来作为基础进行开发。Truffle官方和社区都共同开发了一些样例，这些样例称之为Truffle Boxes ，就和圣诞节礼物一样，一打开盒子就已经是完整的示例。我们首先通过一个名为MetaCoin的项目来了解一下Truffle的工程目录结构。


下载样例
---------------

初始化目录，下载 MetaCoin模板项目：

.. code-block:: bash

   $ mkdir truffle-project
   $ cd truffle-project
   $ truffle unbox metacoin


.. Note::

   Truffle 稳定版本已经去除了truffle init <模板名> 的启动方式，执行 truffle init 会得到一个空白的基础项目；若要下载某模板项目，请使用 truffle unbox <模板名>来进行。

至此我们得到了一个模板项目，它的组织结构如下：

.. code-block:: bash

   truffle-project/
   ├── contracts
   │   ├── ConvertLib.sol
   │   ├── MetaCoin.sol
   │   └── Migrations.sol
   ├── migrations
   │   ├── 1_initial_migration.js
   │   └── 2_deploy_contracts.js
   ├── test
   │   ├── TestMetacoin.sol
   │   └── metacoin.js
   ├── truffle-config.js
   └── truffle.js
   
   3 directories, 9 files

每个目录具体解释如下。
  - contracts目录包含了主要的合约代码，示例中包含了MetaCoin.sol（主要合约），ConvertLib.sol （库文件）和部署时记录地址的合约 Migration.sol。
  - migrations目录包含了两个部署文件，先会部署Migration.sol, 再会链接ConvertLib.sol与MetaCoin.sol部署到链上，两者顺序不可颠倒。代码中aritfacts.require()函数相当于node.js中的require()函数，指明了我们希望引用哪个合约的抽象。这个引用名必须与合约中的类名相同。
  - test目录包含了两种不同的测试文件，智能合约可以通过sol结尾的合约进行测试写作，也可以通过js文件写作，两者功能几乎一致，笔者在开发组多时，考虑到语言的通用性与灵活程度，平时一直使用js的格式书写测试。Truffle的测试都是Mocha测试框架格式的语法。
  - truffle-config.js和truffle.js都保存了控制truffle行为定义的全局变量，当想调换测试网络时，可以编辑truffle.js指定网络和端口。

编译项目
------------

有了目录和源文件，我们来尝试编译部署一下合约。

.. code-block:: bash

   $ truffle compile
   Compiling ./contracts/ConvertLib.sol...
   Compiling ./contracts/MetaCoin.sol...
   Compiling ./contracts/Migrations.sol...
   Writing artifacts to ./build/contracts

Truffle将contracts目录下的合约都编译了一遍，之后将输出结果放入到了./build/文件夹下方，build 文件夹内都是特殊格式的 artifacts，相对应智能合约的 ABI 信息。每次Truffle在跑 ``compile`` 命令的时候，都会创建或者复写这个文件夹，所以修改 build 文件夹的内容是徒劳的，再次编译后，新的改动就会覆盖旧文件。

如果启动 ``--all`` 参数会强制从头编译所有的合约，无论新旧。

.. code-block:: bash

   $ truffle compile --all

部署项目到 Ganache
----------------------------

编译完成后，我们可以将合约部署到网络上。

.. code-block:: bash

   $ truffle migrate

此时可能读者电脑上会报一个错说：没有指定以太坊网络，的确，我们应该先启动 Ganache 测试节点。

.. code-block:: bash

   $ ganache-cli

然后将 truffle.js 文件修改为如下样式，指定测试的网络。

.. code-block:: javascript

   module.exports = {
     networks: {
         development: {
             host: "localhost",
             port: 8545,
             network_id: "*" // 匹配任何的network id
         }
     }
   };

再次运行truffle migrate 就不会出错了，控制台输出如下。

.. code-block:: bash

   Using network 'development'.
   
   Running migration: 1_initial_migration.js
     Deploying Migrations...
     ... 0x3d5fe2cd860c64d465a0fae3908e2de889ba56f4ebd683653f6d23c0eee55761
     Migrations: 0x67b19c3ce017b7b75c4ca1d0b06bd17ec8d31df0
   Saving successful migration to network...
     ... 0xc8829afea94774a85e708608ca631758096e3c9005ab08f5209911ed41cc7c3c
   Saving artifacts...
   Running migration: 2_deploy_contracts.js
     Deploying ConvertLib...
     ... 0x4bc21a0a4df9507b71d1e3cbce199d1eb967b37346156e005bc53c3afd868231
     ConvertLib: 0xbf46ec73895ef7f9bfe52f508bd5041dd3746978
     Linking ConvertLib to MetaCoin
     Deploying MetaCoin...
     ... 0x3326d829e6968d4bb5128f49a0fde951c4d1ed2c8b0bfa4ecaf221fcea38c865
     MetaCoin: 0x7a054ef9375731b2d41beb5cce1336ccebb16d3c
   Saving successful migration to network...
     ... 0x6e4d273dee78e6f807a9958ce24ec717d9081083dacfbf24fadf5880368ae6b1
   Saving artifacts...


测试项目
----------------

在合约部署到测试网上之后，我们可以运行一系列预定义的测试，例如 /test/metacoin.js文件所指定的Mocha测试就可以运行起来，Mocha测试框架配合 Chai 的断言库非常有利于智能合约的测试。以下是运行示例项目的测试结果。

.. code-block:: bash

   $ truffle test
   Using network 'development'.
   
   Compiling ./contracts/ConvertLib.sol...
   Compiling ./contracts/MetaCoin.sol...
   Compiling ./test/TestMetacoin.sol...
   Compiling truffle/Assert.sol...
   Compiling truffle/DeployedAddresses.sol...
   
   
     TestMetacoin
       ✓ testInitialBalanceUsingDeployedContract (81ms)
       ✓ testInitialBalanceWithNewMetaCoin (91ms)
   
     Contract: MetaCoin
       ✓ should put 10000 MetaCoin in the first account
       ✓ should call a function that depends on a linked library (41ms)
       ✓ should send coin correctly (107ms)
   
   
     5 passing (946ms)
   