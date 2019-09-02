:github_url: https://github.com/laalaguer/ethereum-compass/blob/master/ch9/install.rst

编译、测试工具安装
====================

我们来学习一下Truffle这个进阶工具。Truffle [#]_ 在以太坊的开发环境里扮演了举足轻重的角色，作为一个基于JavaScript的Solidity开发框架，Truffle最大特色就是集成了开发与测试一条龙的服务。它同时是一个开发环境、测试框架，也是一个资产管道。它功能强大到让开发者可以不用离开Truffle 命令行即可执行一系列的编译与测试工作。而且它还可以配置对接的以太坊网络，让开发者灵活地选择在各个测试网络之前来回切换。Truffle 拥有的特性有。

  - 内置合约编译、链接、部署管理功能。
  - 对合约可以进行自动化测试。
  - 同时支持公有链、私有链以及测试链，轻松调整测试环境。
  - 完全基于JavaScript编写的项目和API，易于上手。
  - 基于NPM包管理器管理项目，便于添加其他Javascript依赖。
  - 命令行集成度高，方便放置在服务器上做持续集成。

除此之外，Truffle还内置了一个默认的测试底层网络实现 Ganache。Ganache 原名 Test-RPC，是以太坊开源项目中采用JavaScript编写的模拟区块链运行的节点。它本身并不联网产生区块，而是一个测试环境中的高度仿真模拟器。它的Web3命令支持丰富，紧跟社区的最新提案和开发进度，因为没有实际挖矿（仅仅模拟挖矿行为），所以它的运行速度是惊人的，非常利于测试环境下的高速使用，不需要等实际区块链的出块时间间隔。下面是 Ganache 的运行界面。

.. _9-1:
.. figure:: /img/Picture53.png
   :align: center
   :width: 600 px

   Ganache 的UI运行界面，账户界面概览

Truffle的安装
-------------------

通过 npm 管理器我们可以非常轻松地获取Truffle。 Truffle与 Ganache一起安装在全局。安装的前提是，计算机需要 Node.js 高于8.3.0以上的版本。

我们通过如下命令行确认Node.js 的版本：

.. code-block:: bash

   node --version
   v10.9.0

通过如下命令执行最新稳定版 Truffle 的安装：

.. code-block:: bash

   npm install –g truffle
   truffle
   Truffle v4.1.13 - a development framework for Ethereum
   
   Usage: truffle <command> [options]
   
   Commands:
   init      Initialize new and empty Ethereum project
   compile   Compile contract source files
   migrate   Run migrations to deploy contracts
   deploy    (alias for migrate)
   build     Execute build pipeline (if configuration present)
   test      Run JavaScript and Solidity tests
   debug     Interactively debug any transaction on the blockchain (experimental)
   opcode    Print the compiled opcodes for a given contract
   console   Run a console with contract abstractions and commands available
   develop   Open a console with a local development blockchain
   create    Helper to create new contracts, migrations and tests
   install   Install a package from the Ethereum Package Registry
   publish   Publish a package to the Ethereum Package Registry
   networks  Show addresses for deployed contracts on each network
   watch     Watch filesystem for changes and rebuild the project automatically
   serve     Serve the build directory on localhost and watch for changes
   exec      Execute a JS module within this Truffle environment
   unbox     Download a Truffle Box, a pre-built Truffle project
   version   Show version number and exit


我们确认安装的Truffle版本是v4.1.13。值得注意的是，Truffle 在npm安装过程中已经被安装到了全局，而非一个项目中，所以我们可以在控制台非常轻松地直接键入truffle命令来启动Truffle。Truffle相关的使用命令如下表所示，我们最常用的是compile、test两个命令。

+----------+----------------------------------+
| 子命令   | 解释                             |
+----------+----------------------------------+
| init     | 初始化一个项目，可从模板初始化   |
+----------+----------------------------------+
| compile  | 编译一个智能合约项目             |
+----------+----------------------------------+
| migrate  | 执行部署脚本                     |
+----------+----------------------------------+
| deploy   | 执行部署脚本，和 migrate 同义词  |
+----------+----------------------------------+
| test     | 执行指定的测试                   |
+----------+----------------------------------+
| networks | 展示各个以太坊网络部署的合约地址 |
+----------+----------------------------------+
| watch    | 追踪源代码修改，随时重构项目     |
+----------+----------------------------------+
| serve    | 启动本地服务器，展示项目情况     |
+----------+----------------------------------+


Ganache的安装
----------------------

Ganache 继承了原来的 Test-RPC的使命，作为测试环境里的模拟区块链而存在（我们终于可以不使用Geth来建立真实的节点网络了）。我们需要8.3.0以上的Node.js环境支持。我们通过如下命令安装 Ganache。

.. code-block:: bash

   $ node --version
   v10.9.0
   $ npm install -g ganache-cli
   

其实 Ganache 也就是旧的 Test-RPC 项目换了一个名字而已，你同样可以安装 testrpc 来安装同一个软件。

.. code-block:: bash

   $ npm install –g testrpc

启动Ganache与启动testrpc一致，都只需要输入包名即可。

.. code-block:: bash

   $ ganache-cli

   Ganache CLI v6.1.6 (ganache-core: 2.1.5)
   
   Available Accounts
   ==================
   (0) 0xf096552fd17b6d43ab4fc8cde9013c7e2e59e92a (~100 ETH)
   (1) 0x28908951d8621570f21a36dbca8568c7c0403b9a (~100 ETH)
   (2) 0x3dbe5e8338ba08c0e95841d1cd88d280521da167 (~100 ETH)
   (3) 0x4902e5f070a2ad3dd13faaf73fdf5f00af77db06 (~100 ETH)
   (4) 0x27b3df0443a0a653fd28972024a45d4bbf45cd88 (~100 ETH)
   (5) 0xf44c4f3d26ee666ce0c8e25b0cad51cda3038d54 (~100 ETH)
   (6) 0xda0eeefc53c602c40280b6199efe6a2596fc6a83 (~100 ETH)
   (7) 0xc2a2d0116ffc4b088e73a6e1ae7f9c2a9775023a (~100 ETH)
   (8) 0x10c7bea0cbc1b864db044696b1269819696f9c0c (~100 ETH)
   (9) 0xa3c62dcfe700c213a9a2cfb911f40e8b2c8fe983 (~100 ETH)
   
   Private Keys
   ==================
   (0) 0x33270e0fef2525619fdf10b3568abc21a2fed4c2f2c5a597c1be78d898599949
   (1) 0xcdd6263fabbfeef6924ef4fbb2510dee9757f56f5bbae0f54e2ff433a21d6039
   (2) 0x76e99dcf6f7fd9ef2e8ac62f0c9c90a1a94ec9d9aaf5ee995678a007147c7763
   (3) 0x4796fd5c3202fda98ceb0f2d981d732ace2aedcaace50ff865158884698fe5b4
   (4) 0xa43347fbcadfb28af70c8bb2c008da0d279934f88f8930b06dddaabdb7efd4da
   (5) 0x83a1fd902af38b0e2ffac7bcb16bcd21434ef64a24cfd91dd594f350fcb0fbc0
   (6) 0x3102424399c97bbfd7f6d23387f6bd8b8822e168ecc39d5d2faaabe830e1025e
   (7) 0x2e25026d88e2095e7a712a881159ebba55e736b4b822d427af9d06eaefd1d254
   (8) 0xd0be42aefa447ed693daaca4f9b449f55ceb86744efa36f369eab449038e48f7
   (9) 0x06574e68519dd2773a0987aa9f25d2ef8bc8a0c91d88447fad087dba4d247f76
   
   HD Wallet
   ==================
   Mnemonic:      wolf capable soon aisle crunch chair subway prosper fall make grab range
   Base HD Path:  m/44'/60'/0'/0/{account_index}
   
   Gas Price
   ==================
   20000000000
   
   Gas Limit
   ==================
   6721975
   
   Listening on 127.0.0.1:8545

在以上启动过程中，我们发现Ganache已经自动帮我们创建了10个测试用的地址，它们各自对应的私钥也已经打印在控制台上；Ganache还额外为我们模拟了一个硬件钱包以及硬件钱包的助记词；Ganache已经代我们确定了网络的gasPrice和单一区块的gas上限；Ganache启动并监听在我们的端口8545上；Ganache 的版本号是v6.1.6。

至此，Ganache可以完全替代我们早先的Geth客户端作为测试的通讯节点。在我们对代码信心十足后，我们可以将测试网络换成Geth节点的私链进行近似公链的测试，再之后可以部署到以太坊公链的测试网上进行全真测试。


.. [#] 笔者注：项目源代码地址 https://truffleframework.com/