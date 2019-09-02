第 4 章 数据结构
==========================

本章数学相关。讲解以太坊里的数据结构。

以太坊大量使用名为 **Merkle Patricia Trie** （MPT树）的高效结构对数据进行组织、索引。此外，对于树的节点，使用一种高效的键值对数据库 **LevelDB** 进行本地持久化保存。

我们来循序渐进地学习一下。

.. toctree::

   radix.rst
   merkle.rst
   mpt.rst
   rlp.rst
   interval.rst
   stateroot.rst
   transroot.rst
   receiptroot.rst
   block.rst