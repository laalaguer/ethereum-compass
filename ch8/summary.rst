:github_url: https://github.com/laalaguer/ethereum-compass/blob/master/ch8/summary.rst

小结
============

本章我们手把手地带读者完成了Solidity 的语法旅程。在头绪繁多的语法和结构学习中，我们着重介绍了合约的结构，数据结构，继承关系和特色语法--函数修饰符。希望在读者阅读的时候与上一章的虚拟机相配合起来看，温故而知新。

智能合约不仅仅可以执行固定指令，修改区块链上的数据，也可以接受以太币，或者自动地将以太币发送给目标对象。

在实际应用的辽阔领地里，诞生了无数的优秀智能合约，它们在安全性和编写的灵活性上都有极大的提升，开源项目openZeppelin就是其中佼佼者，它的代码经过了安全专家的审计。多数安全编程都在其中摘录需要的代码片段。读者可以在github上搜索它的名字来阅读他人的精华。

在下一章节中，我们将着重介绍使用Truffle工具构建一个生活中实际存在的智能合约--ERC20代币合约，让读者更了解合约编写和部署生态，我们也会介绍SafeMath这个鼎鼎大名的安全数学函数库（openZeppelin）的一部分，用来解决以太坊虚拟机的溢出问题。
