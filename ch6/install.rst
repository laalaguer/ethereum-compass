安装编译器
==================

本章我们通过两种手段来部署同一份智能合约，合约内容非常简单。我们通过 Solidity 编程语言(开发智能合约最流行的语言)来写智能合约。我们需要用到 Solc 编译器(0.4.24版)。

.. topic:: Ubuntu环境下安装 Solc

   同安装Geth一样，我们通过PPA源来安装。

.. code-block:: bash
   :linenos:

   sudo apt-get install software-properties-common
   sudo add-apt-repository ppa:ethereum/ethereum
   sudo apt-get update
   sudo apt-get install solc
   solc --version


.. topic:: Mac环境下安装 Solc

   同安装 Geth一样，我们通过Homebrew包管理器来安装 solc。

.. code-block:: bash
   :linenos:
   
   brew tap ethereum/ethereum
   brew install solidity
   solc --version
   
   solc, the solidity compiler commandline interface
   Version: 0.4.24+commit.e67f0147.Darwin.appleclang
