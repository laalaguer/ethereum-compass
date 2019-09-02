上手实践：ERC20合约测试
===============================

在Truffle的测试中，我们选用 Ganache(Test_RPC)作为底层的区块链节点。它具有良好的通讯与实时反馈能力，而且它的块也是按需打包，每发生交易才会出新的区块（回忆第6章我们反复启动、暂停挖矿的繁琐，Ganache帮我们代劳了这部分工作），避免了资源的浪费。要测试ERC20的合约，我们必须覆盖到全部6个方法，并且进行反复的尝试，试整型溢出、测试授权不足、测试余额不足，并且我们要测试调用失败时是否会返回之前的状态。

准备工作
--------------

Truffle 引用结构都以目录为准，相对比较宽松，我们新建一个helpers目录并加一个文件，用来存放我们测试所用的函数断言的辅助函数；另外我们再添加一个与智能合约同名的JavaScript测试文件作为测试的剧本；此外因为我们程序中的数字都比较大，我们需要额外引用一个大数字的处理库来帮助我们处理大数字加减法。

安装大数字处理库bignumber.js

.. code-block:: bash

   $ npm install --save bignumber.js

添加Revert相关断言助手assertRevert.js，稍后填充具代码。

.. code-block:: bash

   $ mkdir helpers
   $ touch assertRevert.js

添加测试主文件Cat.js

.. code-block:: bash

   $ cd test/
   $ touch Cat.js

至此，我们的项目基本文件已经完整，完整项目结构如下。

.. code-block:: bash

   erc20-test/
   ├── contracts
   │   ├── Cat.sol
   │   ├── ERC20.sol
   │   ├── ERC20Basic.sol
   │   ├── Migrations.sol
   │   └── SafeMath.sol
   ├── helpers
   │   └── assertRevert.js
   ├── migrations
   │   └── 1_initial_migration.js
   ├── node_modules
   │   └── bignumber.js
   │       ├── CHANGELOG.md
   │       ├── LICENCE
   │       ├── README.md
   │       ├── bignumber.d.ts
   │       ├── bignumber.js
   │       ├── bignumber.js.map
   │       ├── bignumber.min.js
   │       ├── bignumber.mjs
   │       ├── bower.json
   │       ├── doc
   │       │   └── API.html
   │       └── package.json
   ├── package-lock.json
   ├── test
   │   └── Cat.js
   ├── truffle-config.js
   └── truffle.js


测试辅助函数与库
------------------------

Mocha框架和Chai断言库（测试中的库）都不是为了以太坊测试而专门设计的，为了在测试中，探测到一笔交易失败后是状态重置了退回了，还是直接异常了扣完了所有的gas费，我们需要有一个revert关键词的判断助手，所以我们编写了assertRevert.js助手函数库。

.. code-block:: javascript

   async function assertRevert (promise) {
     try {
       await promise;
     } catch (error) {
       const revertFound = error.message.search('revert') >= 0;
       assert(revertFound, `Expected "revert", got ${error} instead`);
       return;
     }
     assert.fail('Expected revert not received');
   }
   
   module.exports = {
     assertRevert,
   };

函数代码比较简单，就是在异常后返回的 Promise 值里面判断是否有带revert关键字。

测试代码分析
------------------

我们来逐步构建Cat.js 测试剧本。

.. code-block:: javascript

   const Cat = artifacts.require('Cat')
   const BigNumber = require('bignumber.js')
   const { assertRevert } = require('../helpers/assertRevert');
   
   contract('Cat', function(accounts){
     const symbolName = 'CAT'
     const decimals = web3.toBigNumber(18)
   
     const allTokens = new BigNumber(10000000000e18)
   
     let catInstance
   
     const deployer = accounts[0]
     const user1 = accounts[1]
     const user2 = accounts[2]
   
     beforeEach('setup contract for each test', async function () {
       catInstance = await Cat.new()
     })

在开头我们引用了artifacts.require(‘Cat’)获取到CAT智能合约的ABI，我们就具备了部署它、和它互动的方法。接着引入了bignumber 来处理大数字，还有我们亲自编写的assertRevert库辅助我们检查交易失败时revert关键行为有没有触发。我们用合约关键字contract开头，表示我们要测试一个合约。在接下来我们设立了三个账户：合约部署账户、用户1和用户2（代码中为deployer,user1, user2）。

这里尤其注意的是，我们调用了beforeEach()函数，让每一次测试之前我们都部署一个新的合约，这样有利于合约存储空间的初始化，让每个合约测试互不干扰。

.. code-block:: javascript

   describe('Initla deployment', function(){
     describe('totalSupply', function(){
       it('returns 1 * (10 ** 10) * (10 ** 18)', async function (){
         const total = await catInstance.totalSupply()
   
         assert.equal(total.toString(), allTokens.toString())
       })
     })
   
     describe('symbol', function(){
       it('returns correct symbol name', async function(){
         const symbol = await catInstance.symbol()
   
         assert.equal(symbol, symbolName)
       })
     })
   })

Mocha 框架通过describe关键字来将测试语句分组，每个分组包含了多个子describe 语句。我们在初始化阶段检查Cat合约的符号和发行总量是否符合我们的预期。

.. code-block:: javascript

   describe('init balance', function () {
     describe('user1 has no tokens', function () {
       it('returns zero', async function () {
         const balance = await catInstance.balanceOf(user1)
   
         assert.equal(balance, 0)
       })
     })
   
     describe('deployer has all tokens', function () {
       it('returns the total amount of tokens', async function () {
         const balance = await catInstance.balanceOf(deployer)
   
         assert.equal(balance.toString(), allTokens.toString())
       })
     })
   })

在合约创世后，我们检查构造函数是否如约定将所有的CAT币转移给了创始人 deployer，并检查用户1的账户里面持有CAT币的数量为0。

.. code-block:: javascript

   describe('transfer', function(){
     describe('normal transfers', async function(){
       it('transfers among deployer, user1, user2', async function(){
         // Success: transfer all tokens deployer => user1
         let amount1 = web3.toBigNumber(10000000000)
         let value1 = amount1.times(web3.toBigNumber(10).pow(decimals))
   
         await catInstance.transfer(user1, value1, { from: deployer })
         let deployerBalance = await catInstance.balanceOf(deployer)
         let user1Balance = await catInstance.balanceOf(user1)
   
         assert.equal(deployerBalance, 0)
         assert.equal(user1Balance.toString(), value1.toString())
   
         // Success: transfer 500 tokens user1 => user2
         let amount2 = web3.toBigNumber(500)
         let value2 = amount2.times(web3.toBigNumber(10).pow(decimals))
   
         await catInstance.transfer(user2, value2, { from: user1 })
         let user1Balance_new = await catInstance.balanceOf(user1)
         let user2Balance_new = await catInstance.balanceOf(user2)
         assert.equal(user2Balance_new.toString(), value2.toString())
         assert.equal(user1Balance.minus(user1Balance_new).toString(), value2.toString())
       })
     })
   
     describe('abnormal transfers', async function(){
       it('throws on insufficient balance', async function(){
         // Fail: insufficient funds
         let amount = web3.toBigNumber(1000)
         let value = amount.times(web3.toBigNumber(10).pow(decimals))
   
         await assertRevert(catInstance.transfer(user1, value, { from: user2 }))
         let user1Balance = await catInstance.balanceOf(user1)
         let user2Balance = await catInstance.balanceOf(user2)
   
         assert.equal(user1Balance, 0)
         assert.equal(user2Balance, 0)
       })
     })
   })

在上述代码中，我们对Transfer转账相关的函数进行了深入测试。我们首先进行了一笔正常的转账，将创始人deployer持有的所有CAT币转移给用户1，再将部分CAT币从用户1转移到用户2。

我们接着进行了不正常的转账，从一个余额不足的账户转账出来，这里我们希望引发一场，并且让交易放弃执行，转而触发 revert()函数回到测试开头的状态。

.. code-block:: javascript

   describe('allow', function(){
     describe('normal allow', async function(){
       it('allows user1 to transfer from deployer', async function(){
         let amount = web3.toBigNumber(1000)
         let value = amount.times(web3.toBigNumber(10).pow(decimals))
   
         // Approve transfer
         await catInstance.approve(user1, value, { from: deployer })
         // Check allowance
         let allowance = await catInstance.allowance(deployer, user1, { from: user1 })
         assert.equal(allowance.toString(), value.toString())
   
         // Success: transferFrom
         await catInstance.transferFrom(deployer, user1, value, { from: user1 })
         // Check balance and allowance
         let allowance_new = await catInstance.allowance(deployer, user1, { from: user1 })
         assert.equal(allowance_new, 0)
         let user1Balance = await catInstance.balanceOf(user1)
         assert.equal(user1Balance.toString(), value.toString())
       })
     })
   
     describe('abnormal allow', async function(){
       it('throws on insufficient balance', async function(){
         let amount = web3.toBigNumber(1000)
         let value = amount.times(web3.toBigNumber(10).pow(decimals))
   
         // Success: approve transfer
         catInstance.approve(user2, value, { from: user1 })
         // Fail: cannot transfer
         await assertRevert(catInstance.transferFrom(user1, user2, value, { from: user1 }))
       })
     })
   })
   
   })


我们最后对 approve() 类型的函数进行测试。首先进行了一次正常的授权，并且在授权后调用 transferFrom()函数将该授权兑现，转走一定数量的CAT币。接着我们又进行了异常测试，让授权照常进行，但是账户余额却不足。此时验证transferFrom()会因为余额不足而失败，触发 revert()让账户状态回到测试开始的样子。

至此，一份完整的 Cat.js 测试文件已经呈现在读者眼前，我们将它保存下来，执行测试吧！

测试运行与结果
---------------------

请确保你的 Ganache 节点已经启动

.. code-block:: bash

   $ ganache-cli

代码的测试过程很简洁，仅需要运行一行命令就可以让truffle帮我们编译、部署、测试。

.. code-block:: bash

   $ truffle test
   Compiling ./contracts/Cat.sol...
   Compiling ./contracts/ERC20.sol...
   Compiling ./contracts/ERC20Basic.sol...
   Compiling ./contracts/Migrations.sol...
   Compiling ./contracts/SafeMath.sol...


     Contract: Cat
       Initla deployment
         totalSupply
           ✓ returns 1 * (10 ** 10) * (10 ** 18)
         symbol
           ✓ returns correct symbol name
       init balance
         user1 has no tokens
           ✓ returns zero
         deployer has all tokens
           ✓ returns the total amount of tokens
       transfer
         normal transfers
           ✓ transfers among deployer, user1, user2 (124ms)
         abnormal transfers
           ✓ throws on insufficient balance (54ms)
       allow
         normal allow
           ✓ allows user1 to transfer from deployer (148ms)
         abnormal allow
           ✓ throws on insufficient balance (43ms)

     8 passing (864ms)

总共耗时864毫秒，不到1秒钟时间就完成了测试。在普通的Geth节点上或者以太坊主网上，一个测试就要跑至少15秒，因为要等真实区块的挖掘。一个测试集合，往往需要跑数分钟。Ganache作为测试环境下的节点还是独具优势的。

接下来智能合约已经具备100% 的代码覆盖测试，下一步就是上主网测试，以及发布到主网供全世界的爱好者共同运行了。本书将这个最终发布的问题，留给读者作为家庭作业解决，相信读者从本书中可以找到不止3种部署该合约到以太坊主网的方法。




