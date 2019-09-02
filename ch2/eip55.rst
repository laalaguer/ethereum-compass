:github_url: https://github.com/laalaguer/ethereum-compass/blob/master/ch2/eip55.rst

资料篇：EIP-55 格式的账户地址
==========================================

**哦不，我的转账地址好像填错了！**

你不是第一个碰上问题的人。在钱包的实际使用过程中，以太坊的用户发现，由于缺乏类似比特币的自带地址校验机制，以太坊的公开地址在当朋友之间转账时，经常因为误输入字符而转错了账号。

社区用户在以太坊很早期的时候，就已经提出该问题并思考改进的方法，经过整理成文，形成了 :guilabel:`EIP-55` 提议 [#]_。

该提议按照一定逻辑，将地址中的部分字母大写，与剩余的小写字母来形成校验和，让地址拥有自校验的能力，具体的代码如清单2-5所示。

.. code-block:: javascript
   :caption: 代码清单2-5

   const createKeccakHash = require('keccak')

   function toChecksumAddress (address) {
     address = address.toLowerCase().replace('0x', '')
     var hash = createKeccakHash('keccak256').update(address).digest('hex')
     var ret = '0x'

     for (var i = 0; i < address.length; i++) {
       if (parseInt(hash[i], 16) >= 8) {
         ret += address[i].toUpperCase()
       } else {
         ret += address[i]
       }
     }

     return ret
   }

下表对列举了带校验的地址和普通的地址的对比。这种校验方式可以与旧版本的20个字节的普通地址保持长度一致。

:guilabel:`EIP-55` 方案在地址中平均生成 15 个检查点，将错误输入未能被校验的失误率降到 0.0247% 以下。
目前该地址格式已经被客户端广泛采用，并显著减少了转账出错的情况。

+-------------------------+--------------------------------------------+
| 带校验地址 (大小写区分) | 0x7c52e508C07558C287d5A453475954f6a547eC41 |
+-------------------------+--------------------------------------------+
| 普通地址 (全小写)       | 0x7c52e508c07558c287d5a453475954f6a547ec41 |
+-------------------------+--------------------------------------------+

.. Note::
   Javascript代码可以参考笔者的Github项目：http://github.com/laalaguer/VeChain-Address/

.. [#] Vitalik Buterin, Alex Van de Sande, (2016) ‘Mixed-case checksum address encoding’, Available at: https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md