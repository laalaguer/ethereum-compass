:github_url: https://github.com/laalaguer/ethereum-compass/blob/master/ch4/mpt.rst

.. _reference-mpt:

Merkle Patricia树
===============================

区块链中的数据往往需要满足大量的节点查询、插入、删除、真伪鉴定及存在性检查。

以太坊团队开创性地融合了Radix树和Merkle树的结构，并进一步优化该结构的存储性能，形成了实用的 **Merkle Patricia树** 。

Merkle Patiricia树 (以下简称MPT树) 是一个可以存储键值对 (key-value pair) 的高效数据结构，键、值分别可以是任意的序列化过的字符串。它具有Merkle树的密码学安全校验功能，提供树中的数据完整性与存在性的 Merkle 证明。它具有强大的互动能力，在log(N)级别的时间内完成插入、删除、查询等操作。我们来循序渐进地查看如何构建一棵 MPT树。

.. topic:: 首先我们定义一个节点。

   节点在逻辑上，由 :guilabel:`索引` 、:guilabel:`路径` 和 :guilabel:`数据` 这三部分组成。
   
   在实际中, :guilabel:`索引`\ 对应了数据库软件中一条记录的 `键`；将 :guilabel:`路径` 和 :guilabel:`数据` 编码成一个单元，称为 `存储`，对应了数据库软件中一条记录的“值”。

   节点可以利用数据库软件例如MySQL、SQLite或轻量级数据库LevelDB来存储，它需要保证查询一个索引时候快速高效。
   
   以太坊节点的 :guilabel:`路径` 是16格的Hex值，表示为0~f 的16个元素，节点的第17个元素是节点存储。具体结构如下图 4-7_ 所示, 数据库存储 `key` 和 `value` 为一组键值对。其中 `value` 包含了 :guilabel:`路径` 和 :guilabel:`数据` 。

.. _4-7:
.. figure:: /img/Picture33.png
   :align: center
   :width: 600 px

   以太坊的节点格式： 哈希值为索引，节点对应值为路径和节点存储

.. Note::
   请不要将索引(key)与路径(Path)混为一谈。

   数据库“眼中”每一组数据 = 索引+存储， 而存储 = 路径+数据。

   路径存在的意义是在组织数据的时候与其他数据关联的查找方式。
   
   在以太坊中，数据库查找，我们简称为 **dl操作(Database lookup)** 。树路径查找，简称为 **tl操作(Trie lookup)** 。

.. topic:: 初步构建一棵低效率的MPT树

   假设我们拥有如下的键值对数据集。我们想办法用MPT树将其组织起来。

.. code-block:: javascript
   :caption: 一组假想的数据

   {
     'cab8': 'dog',
     'cabe': 'cat',
     '39': 'chicken',
     '395': 'duck',
     '56f0': 'horse'
   }

我们仔细观察数据，首先想到的是Radix树的概念。``cab8`` 和 ``cabe`` 共享了 ``cab`` 这个公共路径，``39`` 和 ``395`` 共享了 ``39`` 这个路径。 ``56f0`` 因为和其他人没有重叠，所以自己单独一个路径。我们可以将这些路径按照字符拆开来，放入节点的 :guilabel:`路径` 中。

其次是诸如 ``"dog"``, ``"cat"`` 之类的数据，它们千变万化，而且往往是一大块。不适合拆分，适合集中存放。我们将其放入节点的 :guilabel:`数据` 部分。

最后是校验和。因为我们希望这组数据作为一个整体，无论添加、删除、变更，都能立即体现出来。所以势必引入一个Merkle树的概念，对每层数据进行哈希，这是额外的功夫，这些哈希结果，放入 :guilabel:`索引` 部分。

具体拆分后是怎样存储的MPT树呢？见图 4-8_。

.. _4-8:
.. figure:: /img/Picture34.png
   :align: center
   :width: 600 px

   一棵低效率的 MPT 树，树中有很多空白值

我们试着查找 :guilabel:`路径` ``56f0`` 对应的数据:

  - 拆分 ``56f0`` 为 ``5、6、f、0`` 四格。
  - 我们首先确认这条数据位于一个数据集，这个数据集的 :guilabel:`索引` 是 *rootHash* 。
  - 我们先做一次数据库查询找到该节点 (dl操作)。找到后，我们从这一行开始。
  - 第一段路径为 ``5`` ，我们取出 :guilabel:`路径` 顺序找到第5格，查询得知对该格对应的 :guilabel:`索引` *HashE* (tl操作)。
  - 根据 *HashE* 再次进行数据库查询 (dl操作)，得到E节点，E节点不包含 :guilabel:`数据`，所以它肯定是个叶子节点。
  - 第二段路径为 ``6`` ，我们路径查询 (tl操作) 得知对应 :guilabel:`索引` *HashF*。
  - 以此类推，依序找到 *HashG* 、 *HashH* 。
  - 取出 *HashH* 对应节点的 :guilabel:`数据` ，即为 ``“horse”`` ，查找完毕。

在该过程中我们交替使用了 **数据库查找** 和 **路径查找** 。
路径查找是Radix树的特色。把节点进行哈希进行索引，就是Merkle树的特色。两者完美结合。

但是这棵树还不够优化，存储空间有大量的空白值， *HashE、HashF、HashG* 的节点仅仅指向一个后继节点，在空间上效率不高，产生了 **退化** 。

.. topic:: 空间效率改进的MPT树

   我们下面来改进一下这棵树，将部分节点改造合并、缩短路径，如图 4-9_ 所示。

.. _4-9:
.. figure:: /img/Picture35.png
   :align: center
   :width: 600 px

   存储空间改造后的 MPT 树


上述改造中我们引入了部分新的规则，节点不再是统一的格式，而是分化成了四种格式。

  - 空节点(**null**)：没有包含任何元素，用空字符串表示。
  - 分支(**branch**)：包含 ``17`` 个元素，前 ``16`` 个为 :guilabel:`路径` ，末一个元素为 :guilabel:`数据` 。
  - 叶子(**leaf**)：仅包含 ``2`` 个元素，一个 :guilabel:`路径` 与一个 :guilabel:`数据` 。
  - 扩展节点(**extension**)：仅包含 ``2`` 个元素，一个 :guilabel:`路径` 和 :guilabel:`索引` 。

hashB 是一个 **扩展节点** ，它包含了部分路径 ``{ab}`` 与哈希存储 hashJ，这个扩展节点不是树的终点，因为它不包含可以返回的值，它明确指引查找者往下再去 hashJ 对应的节点查找值数据；

hashE 就是典型的 **叶子节点**，它包含了部分路径 ``{6f0}`` 与最终值 ``“horse”`` ，它再也没有后继节点可供再查找下去；

hashC 是一个 **分支** ，它与叶子节点相似，包含了一个最终值 ``"chicken"`` ，但是也拥有一个对于 hashD 的索引，它不是树的终点，还可以往下继续查找下去。

根节点，分支节点，扩展节点，叶子节点的简化关系如图 4-10_ 所示。

.. _4-10:
.. figure:: /img/Picture36.png
   :align: center
   :width: 600 px

   MPT树：根节点、分支节点、叶子节点、扩展节点和最终数据存储

.. topic:: 在MPT树中查找元素
   
   当我们在MPT树中查找路径 ``56f0`` 对应的 ``"horse"`` 值时，路径就大大缩短，按照如下步骤就可找到。

     - 将查找总路径 ``56f0`` 分段为 ``5、6、f、0`` 。
     - 从根节点 :guilabel:`索引` *rootHash* 开始，我们先做一次数据库查询找到根节点(dl操作)。
     - 第一个路径为 ``5`` ，我们 :guilabel:`路径` 查询得知对应 :guilabel:`索引` *HashE* (tl操作)。
     - 根据 *HashE* 再次进行数据库查询，得到 E 节点。
     - E 节点已经包含了剩下的部分路径 ``{6f0}`` ，命中，取出数据 ``“horse”``。

   在以太坊 MPT 树中，除以上规则外，还有一些额外的规则需要遵守。

     - 每个部分路径都需要执行Hex Prefix编码(Hex Prefix encoding) [#]_ 。
     - 每个节点的元素都需要执行RLP编码(RLP encoding) [#]_ 。
     - 每个节点也需要执行RLP编码 (见图 4-11_ ）。

.. _4-11:
.. figure:: /img/Picture37.png
   :align: center
   :width: 600 px

   以太坊的 MPT 树中的节点 需要进行 RLP 编码

.. [#] Ethereum Community Authors (2019), ‘Hex Prefix Encoding’, The Ethereum Wiki, Available at: https://github.com/exthereum/hex_prefix
.. [#] Ethereum Community Authors (2018), ‘Recursive Length Prefix Encoding’, The Ethereum Wiki, Available at: https://github.com/ethereum/wiki/wiki/RLP
