高级语法和概念
=====================

到现在这步，读者对于智能合约的基本编程已经有了一个充分的了解。相信萦绕在读者心头一定有疑问：以太坊智能合约编程，究竟和普通的无输入输出程序有何区别？在合约里，我如何加入更多的控制结构来保护方法不被恶意调用？本节将逐条讲述各种小技巧以及安全措施，来帮助读者构建更安全的智能合约。

Contract 构造函数
-------------------------

构造函数在以前旧版本的Solidity里面是一个自定义的同名函数，例如下面的合约。

.. code-block:: solidity

   contract Ownable {
     address public owner;
     // 初始化，设定 owner
     function Ownable() public {
       owner = msg.sender;
     }
   }


这里Ownable构造函数对应了Ownable合约，完成了初始化的操作。但是这个旧版的语法在新的编译器里面被替换，因为往往开发者在开发时候更换了合约的名字而忘记修改构造函数名，造成重大事故。所以现在统一用constructor关键字来替代。

.. code-block:: solidity

   contract Ownable {
     address public owner;
     // 初始化，设定 owner
     constructor() public {
       owner = msg.sender;
     }
   }


Ownable控制
-----------------------

控制合约的所有权，常常使用的一个控制结构就是Ownable，这只不过是一个约定俗成的表达写法。下面是openZeppelin公开项目中摘录的部分代码

.. code-block:: solidity

   contract Ownable {
     address public owner;
     event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);
   
     // 初始化，设定 owner
     function Ownable() public {
       owner = msg.sender;
     }
   
     // 设定仅允许 owner 访问
     modifier onlyOwner() {
       require(msg.sender == owner);
       _;
     }
   
     // 转移 owner 所有权
     function transferOwnership(address newOwner) public onlyOwner {
       require(newOwner != address(0));
       emit OwnershipTransferred(owner, newOwner);
       owner = newOwner;
     }
   }


读者可能对onlyOwner()函数看得云里雾里。这里怎么会有一个 ``_;`` 奇怪的段落呢？其实这里 _; 函数代表了模板预留空位，当任意新的函数使用onlyOwner来修饰自己的时候，它就可以将自己的函数逻辑代码替代到 _; 位置来执行。例如下所示。

.. code-block:: solidity

   contract MyContract is Ownable {
     event Laugh(string laughter);
   
     // 用 OnlyOwner来修饰函数
     function likeABoss() external onlyOwner {
        emit Laugh("Muahahahaha");
     }
   }

这里函数修饰符除了public/private, internal/external以外，还用onlyOwner来修饰，所以当任何区块链用户试图调用likeABoss()函数的时候，如果不是合约的合约所有者（owner），则无法触发这个函数执行。这样就可以保留合约的所有者对于合约中部分函数的排他操控性。

Pausable控制
--------------------

有了Ownable来明确合约所有权，那么合约是否可以由合约所有者（owner）控制，随时的暂停呢？因为有些合约是只能转账合约，它一旦发生灾难则暴露在公众下不可控制，如果合约所有者能够进行“临时暂停”该多好啊！Pausable可暂停合约就是这样设计的。

.. code-block:: solidity

   contract Pausable is Ownable {
     event Pause();
     event Unpause();
   
     bool public paused = false;
     
     modifier whenNotPaused() {
       require(!paused);
       _;
     }
   
     modifier whenPaused() {
       require(paused);
       _;
     }
   
     function pause() public onlyOwner whenNotPaused {
       paused = true;
       emit Pause();
     }
   
     function unpause() public onlyOwner whenPaused {
       paused = false;
       emit Unpause();
     }
   }


这里的代码一目了然：合约继承自ownable, 所以就有了合约所有权的确立；代码可以执行pause()和unpause()两种来进行暂停/继续。两种函数whenNotPaused()和 whenPaused()可以进行函数修饰，分两种情况，让合约函数可以被调用或者不能调用。

.. admonition:: 小练习

   请对下列合约的代码进行填补，让他在合约暂停时无法执行转账函数transfer。
   
   .. code-block:: solidity
   
      contract Token is ______ {
        event Transfer(address from, address to);
      
        function transfer(address _to) public _______ {
          emit Tranfer(msg.sender, _to);
        }
      }


省钱妙招：struct 结构体
---------------------------------

因为运行一次以太坊智能合约都会花费用户真金白银，所以针对以太坊合约的优化也是编程的关注重点。一份好的智能合约不但在步骤上能节约资源消耗，在存储空间上也是寸土必争。我们可以通过 struct结构体来构建更加空间紧凑的存储，进而节约花费。

回顾一下，我们已知有四种数据结构 uint(uint256)、uint32、uint16和uint8。一般而言单独使用的话，并没有任何空间优化的必要，因为Solidity编译器目前还是用256位来统一存储单值。优化反而对程序可扩展性进行了限制。但是在结构体中优化是很有必要的，例如下面的两个结构体Normal和Mini。

.. code-block:: solidity

   struct Normal {
     uint a;
     uint b;
     uint c;
   }
   
   struct Mini {
     uint32 a;
     uint32 b;
     uint c;
   }


上面的Normal结构能够占领3x256位的空间，然而下面的Mini可以把a、b两者放入同一个256位空间内，c单独放入256位空间。实现了2x256就能存储的优化。在常年累月的运行中，可以完美节约1/3的存储花费！

.. admonition:: 小练习

   请将下列每个数据结构，改为 uint32，并简述可以节约多少空间。

   .. code-block:: solidity

      struct Normal {
        uint a;
        uint b;
        uint c;
        uint d;
      }

时间单位表达
---------------------

在智能合约中经常使用到时间信息。转账时间、合约的倒计时、方法调用的冷却时间都会用到时间。除了用区块链的高度来“估算”时间以外（因为出块时间相对稳定），Solidity提供了一个Unix时间戳来记录时间，类似于1515527488这样的数字。在编程的时候并不需要我们手动拼写这个数字，仅需要通过文字描述即可达到目的。

.. code-block:: solidity

   function fiveMinutesHavePassed() public view returns (bool) {
     return (now >= (lastUpdated + 5 minutes));
   }

这里now和5 minutes都属于时间表示，我们甚至可以直接将时间单位赋值给uint类型的变量。以下的语句是完全合法的。

.. code-block:: solidity

   uint lastUpdated = now;

.. admonition:: 小练习

   请根据提示填充如下的时间单位赋值：

   .. code-block:: solidity

      uint one_day = 1 ____;
      uint five_minutes = ____ mintues;
      uint now_time = ____;
      uint two_hours = __ _______;

带参数的函数修饰符
---------------------------

我们经常看到函数的修饰符可以带一定的参数。这对修饰符号的扩展有很大好处。例如我们可以规定开车的基准年龄不得低于16岁。此时我们可以事先写好一些年龄现值函数，到了最终代码总装配的时候灵活填入16岁这个数值就可以。例如如下的函数。

.. code-block:: solidity

   // ID 和年龄的映射
   mapping (uint => uint) public age;
   
   // 一个函数修饰
   modifier olderThan(uint _age, uint _userId) {
     require(age[_userId] >= _age);
     _;
   }
   
   // 限制年龄大于16岁才能开车
   function driveCar(uint _userId) public olderThan(16, _userId) {
     
   }

这里看到olderThan函数和 driveCar函数其实有一点耦合了。因为olderThan函数需要读取age映射。但是 olderThan函数可以抽象一下，进一步不依赖于任何状态变量，仅仅依赖函数参数输入。这怎么做？可以交给读者进行小练习思考一下。

for 循环
----------------

循环在智能合约中使用范围不是很明显。因为循环消耗大量的计算步骤，智能合约编纂者都尽量减少循环次数，甚至从根本上避免循环的产生。例如下面就是一个循环的例子。

.. code-block:: solidity

   function getOdds() pure external returns(uint[]) {
     uint[] memory odds = new uint[](5);
     
     uint counter = 0;
    
     for (uint i = 1; i <= 10; i++) {
       if (i % 2 != 0) {
         odds[counter] = i;
         counter++;
       }
     }
     return odds;
   }

上述合约代码的思路清晰明了：它首先是一个外部可调用函数，用pure修饰，代表完全没有任何读取区块链的行为。它通过循环for来执行从1到10的遍历，最终输出所有的1-10之间的奇数。


合约收款：payable修饰符
--------------------------------

在虚拟机章节里面提到，以太坊只能合约调用就是消息的传递，消息传递包含两部分：调用智能合约方法（传递数据）以及以太币的支付。那么，合约能否接受调用者的以太币支付呢？答案当然是可以的，请看下列网上商城的代码。

.. code-block:: solidity

   contract OnlineStore {
     function buySomething() external payable {
       // 查看付款金额
       require(msg.value == 0.001 ether);
       // 将对应金额的物品转移给调用方
       transferSomething(msg.sender);
     }
   }

上面多了一个关键字payable和两个环境变量：msg.value和msg.sender。payable表示调用者可以在调用该方法时附上想发送的以太币数量；msg.value则表示实际使用中调用者支付的以太币，这里要和gas费用相区别，gas费用是付给矿工来执行合约的，而 msg.value则是直接付给智能合约的费用。在智能合约收到以太币后，可以直接调用其他函数进行等价交换。这种函数经常在ICO的代币发行中使用，让广大用户可以直接通过合约交换以太坊为代币，无需人工干预，公平、公正、公开。


.. admonition:: 小练习


   请填充下列合约，让用户可以用0.1以太币的价格购买1个 dog 代币。
   
   .. code-block:: solidity
   
      contract OnlineStore {
        mapping (____ => unit) public dogs;
      
        function buyDog() external payable {
          // 查看付款金额
          require(msg.value == ____ ether);
          // 将对应金额的dog转移给调用方
          _____[msg.sender] += 1;
        }
      }

支付费用：transfer方法
-------------------------------

智能合约的作者往往能获得报酬。怎么获得呢？就是通过用户不断向智能合约打入以太币，并积累在智能合约的balance余额中。当智能合约所有者想提取的时候，调用一个函数就可以执行，例如下面所示的合约。

.. code-block:: solidity

   contract GetPaid is Ownable {
     function withdraw() external onlyOwner {
       owner.transfer(this.balance);
     }
   }

这份代码里非常简单。它用this.balance来指代当前合约的总以太币余额。调用了owner.transfer函数来执行以太币的发送，这里owner是一个地址。

.. admonition:: 小练习

   下面的代码片段会将用户购买物品时候的多余以太币退回去，请填充一个函数名。
   
   .. code-block:: solidity
   
      function buy() external  {
        uint itemFee = 0.001 ether;
        msg.sender.__________(msg.value - itemFee);
      }
