:github_url: https://github.com/laalaguer/ethereum-compass/blob/master/ch5/install.rst

安装
===========

.. topic:: Windows环境下安装Geth

   Windows下可以根据操作系统选择64位或者32位的安装包，我们用的Geth的下载包是1.8版本的，在官方下载页面 [#]_ 包含exe 格式的二进制安装文件，下载后双击打开按照默认路径安装好即可使用。下载页面如下图所示：

.. _5-1:
.. figure:: /img/Picture43.png
   :align: center
   :width: 600 px

   以太坊 Geth 客户端下载页面

.. topic:: Ubuntu环境下安装Geth

   在Ubuntu发行版下安装Geth的最快方法是通过内置的 PPA(Personal Package Archives) 软件源进行安装。以太坊为 Ubuntu 发行版 trusty, xenial, zesty, artful 都预备了一个源，仅需通过如下命令添加软件源。

.. code-block:: bash
   :linenos:

   sudo apt-get install software-properties-common
   sudo add-apt-repository -y ppa:ethereum/ethereum

接着通过软件源来安装稳定版本的客户端：

.. code-block:: bash
   :linenos:

   sudo apt-get update
   sudo apt-get install ethereum

若具有先锋精神，想尝试最新版本的非稳定客户端，请安装：

.. code-block:: bash
   :linenos:

   sudo apt-get update
   sudo apt-get install ethereum-unstable

Ubuntu环境下也可以轻松通过源代码编译来安装客户端，Geth 客户端编译依赖 **Golang** 语言环境，官方提供了一个临时项目来安装相应编译依赖并在结束后自动清理项目依赖，运行如下命令执行编译安装：

.. code-block:: bash
   :linenos:

   git clone https://github.com/ethereum/go-ethereum.git
   cd go-ethereum
   make geth

编译后的geth客户端不依赖任何其他文件，是一个可独立运行的命令行软件。

.. topic:: MacOS环境下下安装Geth

   在 Mac OS 下有一个程序员常用的软件源管理器 Homebrew [#]_ ，该管理器可以代为管理 Mac 上的开发环境依赖与包安装。仅需两行命令即可完成安装。

.. code-block:: bash
   :linenos:

   brew tap ethereum/ethereum
   brew install ethereum

安装后检查安装位置与版本.

.. code-block:: bash
   :linenos:

   which geth
   > /usr/local/bin/geth
   geth version
   >
   Geth
   Version: 1.8.14-stable
   Architecture: amd64
   Protocol Versions: [63 62]
   Network Id: 1
   Go Version: go1.10.3
   Operating System: darwin
   GOPATH=
   GOROOT=/usr/local/Cellar/go/1.10.3/libexec

在将来，如果读者想升级Geth，请使用命令：

.. code-block:: bash
   :linenos:

   brew update
   brew upgrade ethereum

.. topic:: Docker镜像安装Geth

   熟悉后端开发的读者也可以通过Docker 镜像的方式安装以太坊客户端Geth，在安装完毕Docker基础环境后，再执行Geth镜像获取；安装过程和普通获取镜像方式相同，运行下列命令即可获取最新版本的客户端。

.. code-block:: bash
   :linenos:

   docker pull ethereum/client-go:latest
   docker run –d –name ether-node –p 8545:8545 –p 30303:30303 \
	ethereum/client-go –fast –cache=512

若想安装其他版本或者旧版本的程序，也可以根据需求调整镜像参数。

  - ethereum/client-go:latest 最新的开发版。
  - ethereum/client-go:stable 最新的稳定版。
  - ethereum/client-go:{version} 某版本号的稳定版。
  - ethereum/client-go:release-{version} 某版本号的稳定发行版。

.. topic:: Node.js的安装

   虽然以太坊等区块链项目本身不是用JavaScript编写的，但在与区块链节点交互的场景中，却大量使用了JavaScript作为接口调用的语言。目前JavaScript的相关的开源软件包的发布、分发、编写都是通过Node.js的包管理器NPM来实现的。我们在使用以太坊的 ``Web3`` 时也选用了NPM项目包管理器来下载、管理我们的 ``web3.js`` 库依赖。例如，在 Mac 下可以通过如下命令安装。

.. code-block:: bash
   :linenos:

   brew install node
   node --version
   > v10.9.0
   npm --version
   > 6.4.0

本书接下去的章节都将围绕Linux/Mac的命令行开发环境讲解，读者需要一台配备了 Linux 操作系统或者Mac操作系统的电脑，并且已经妥善安装好了 Node 与 Geth。


.. [#] 笔者注：下载地址 https://ethereum.github.io/go-ethereum/downloads/
.. [#] 笔者注：下载地址 https://brew.sh/