# 《以太坊的指南针》

### 编者按

**2025-10:** 距离以太坊 1.0 主网上线已经过去了 10 年。本书所列的内容也垂垂老矣。自以太坊 2.0 Beacon 上线以来，PoS的以太坊发生了很多变化，本书内容为 1.0 时代而设，故不再增改。

- 直接查阅：DOCS 文件夹。
- 编译本书：SRC 文件夹。

### 正文

这是给 **程序员** 读的入门书，更是一本面向 **从未接触过以太坊人群** 的书。

以太坊是什么？怎么了解相关知识？

这本书涵盖了你想要了解的以太坊的基本知识。

前半本文艺，后半本实践，附录深刻。

内容包括钱包、交易、数据结构。也包括了Solidity语法、虚拟机原理和实战合约编写。

市面上目前的大部分公链/联盟链都借鉴了以太坊的虚拟机。读完本书，你再去看其他项目，会感到成竹在胸。

书中的知识点都是作者在区块链工作中实践总结的。希望对你们有帮助。

## 目录
<ul>
<li>如何学习这本书</li>
<li>第 1 章 以太坊：一台全球计算机<ul>
<li>简史</li>
<li>发展阶段</li>
<li>以太坊的特色</li>
</ul>
</li>
<li>第 2 章 账户是什么<ul>
<li>小白基础知识问答<ul>
<li>我的以太币记录在哪里？</li>
<li>我的以太币余额如何变化？</li>
<li>什么是区块？</li>
<li>区块和状态的关系</li>
<li>“巨大的账本”</li>
<li>我如何参与以太坊？</li>
<li>我如何与其他人同步账本？</li>
</ul>
</li>
<li>账户探秘<ul>
<li>账户与账户状态</li>
<li>账户状态的内涵<ul>
<li>已执行交易总数</li>
<li>持币数量</li>
<li>存储区的哈希值</li>
<li>代码区的哈希值</li>
</ul>
</li>
<li>没有钱包App, 如何生成账户？</li>
<li>智能合约地址的生成</li>
</ul>
</li>
<li>扩展阅读</li>
<li>资料篇：Keystore 与私钥保存</li>
<li>资料篇：常用钱包 App</li>
<li>资料篇：EIP-55 格式的账户地址</li>
</ul>
</li>
<li>第 3 章 交易是驱动力<ul>
<li>交易的发送<ul>
<li>交易与消息的区别</li>
<li>交易的特性是什么？</li>
</ul>
</li>
<li>交易的样子</li>
<li>交易的生命周期</li>
<li>扩展阅读</li>
<li>资料篇：共识与工作量证明<ul>
<li>比特币的PoW机制（简单版）</li>
<li>比特币算力的中心化问题</li>
<li>以太坊的Pow/Pos机制</li>
</ul>
</li>
<li>资料篇：矿工与挖矿奖励</li>
</ul>
</li>
<li>第 4 章 数据结构<ul>
<li>Radix树</li>
<li>Merkle树和 Merkle证明</li>
<li>Merkle Patricia树</li>
<li>RLP编码<ul>
<li>RLP字符/字符串编码</li>
<li>RLP字符/字符串解码</li>
<li>RLP数组编码</li>
<li>RLP数组解码</li>
</ul>
</li>
<li>扩展阅读</li>
<li>资料篇：状态树 (以及存储树)</li>
<li>资料篇：交易树</li>
<li>资料篇：收据树</li>
<li>资料篇：区块</li>
</ul>
</li>
<li>第 5 章 构建一条以太坊私链<ul>
<li>安装</li>
<li>Geth客户端的结构</li>
<li>启动一条以太坊私链</li>
<li>接收挖矿奖励</li>
<li>转账与收款</li>
</ul>
</li>
<li>第 6 章 手把手教你部署智能合约<ul>
<li>什么是智能合约？</li>
<li>安装编译器</li>
<li>Solc编译智能合约</li>
<li>智能合约发布准备</li>
<li>部署智能合约</li>
<li>调用智能合约</li>
</ul>
</li>
<li>第 7 章 以太坊虚拟机探秘<ul>
<li>虚拟机的执行结果</li>
<li>虚拟机的执行资源</li>
<li>合约调用合约?</li>
<li>虚拟机的输入输出</li>
<li>Gas 花费与退回</li>
<li>虚拟机指令集</li>
</ul>
</li>
<li>第 8 章 Solidity语法练习<ul>
<li>基础概念<ul>
<li>没有浮点数运算</li>
<li>合约基础</li>
<li>变量类型</li>
<li>运算符号</li>
<li>结构体 Struct</li>
<li>数组array</li>
<li>函数申明</li>
<li>类型转换与内置函数</li>
<li>合约与事件</li>
</ul>
</li>
<li>语法进阶<ul>
<li>数据结构：map</li>
<li>环境变量：msg.sender</li>
<li>require还是assert?</li>
<li>继承和引入</li>
<li>省钱妙招：内存变量</li>
<li>接口与合约调用</li>
<li>多返回值</li>
</ul>
</li>
<li>高级语法和概念<ul>
<li>Contract 构造函数</li>
<li>Ownable控制</li>
<li>Pausable控制</li>
<li>省钱妙招：struct 结构体</li>
<li>时间单位表达</li>
<li>带参数的函数修饰符</li>
<li>for 循环</li>
<li>合约收款：payable修饰符</li>
<li>支付费用：transfer方法</li>
</ul>
</li>
<li>小结</li>
</ul>
</li>
<li>第 9 章 Truffle合约开发实战<ul>
<li>编译、测试工具安装<ul>
<li>Truffle的安装</li>
<li>Ganache的安装</li>
</ul>
</li>
<li>Truffle启动样例项目<ul>
<li>下载样例</li>
<li>编译项目</li>
<li>部署项目到 Ganache</li>
<li>测试项目</li>
</ul>
</li>
<li>上手实践：ERC20合约<ul>
<li>新建项目目录</li>
<li>ERC20 Basic合约接口</li>
<li>ERC20 合约接口</li>
<li>SafeMath基础数学库</li>
<li>猫币：CAT数字资产合约</li>
</ul>
</li>
<li>上手实践：ERC20合约测试<ul>
<li>准备工作</li>
<li>测试辅助函数与库</li>
<li>测试代码分析</li>
<li>测试运行与结果</li>
</ul>
</li>
</ul>
</li>
<li>附录 有意思的冷知识<ul>
<li>短地址攻击</li>
<li>比特币的区块</li>
<li>以太坊与比特币账户的区别<ul>
<li>隐私与安全性的比较</li>
<li>数据体积与并发能力</li>
<li>发送交易时对双花的处理</li>
</ul>
</li>
<li>“不可能的三角”问题</li>
<li>ETHASH 挖矿算法<ul>
<li>ETHASH和比特币PoW的异同</li>
<li>ETHASH的设计目标</li>
<li>ETHASH的挖矿运行总流程</li>
<li>ETHASH算法源代码解读</li>
</ul>
</li>
</ul>
</li>
</ul>