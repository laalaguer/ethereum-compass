:github_url: https://github.com/laalaguer/ethereum-compass/blob/master/ch8/levelup.rst

语法进阶
===========

本小节我们进一步讲解常用的合约与合约的关系，并引入更高级的数据结构。 


数据结构：map
---------------------

我们已经学习过了结构体struct和数组两种高级数据结构，这两者都是为了有结构地存储数据而设计的。另一种在编程语言中不可或缺的数据结构是映射关系。。在Solidity语言中也如同Python的dict或者JavaScript的对象一样，现成内置了一个映射数据结构，mapping。

例如我们可以存储账号地址以及它对应的合约内的token数量的关系(假设是一个代币合约)。

.. code-block:: solidity

   mapping (address => uint) public accountBalance;

这里我们申明关键字mapping表示这是一个映射关系；address代表的是一个账户的地址；uint代表了该账户对应的token数量；接着我们用修饰符public申明这个变量可以读取；这个映射关系我们命名为accountBalance。

.. admonition:: 小练习

   请申明两个 mapping 映射。

   保存汽车Car和对应的主人Owner address的对应关系。

   保存主人地址address和它对应的 Car 的数量关系。
   
   .. code-block:: solidity

      Car[] public cars;
      mapping (uint => _____) public carToOwner;
      mapping (_____ => uint) ownerCarCount;


环境变量：msg.sender
-----------------------------

智能合约没有main函数，它是被动地响应外部调用的。谁来调用呢？根据之前我们第7章对以太坊虚拟机的学习，可以知道肯定是首先由外部使用者触发的，这种触发可以调用一个合约，该合约也可以链式调用其他合约为自己填充部分数据、执行某项操作。调用的当事人地址可以被智能合约知晓，即为msg.sender全局变量。这个全局变量存在于每个智能合约的执行环境上下文中。

.. code-block:: solidity

   mapping (address => uint) myNumber;

   function setMyNumber(uint _myNumber) public {
     // 设置一个调用者最喜欢的数字
     myNumber [msg.sender] = _myNumber;
   }
   
   function whatIsMyNumber() public view returns (uint) {
     // 取回设置好的数字，如果未被设置，则返回0
     return myNumber [msg.sender];
   }

上述合约代码片段，允许调用者设置一个数字，并且把数字再次从合约中读取出来。在这两个操作中都直接对于一个映射对象myNumber进行了读写操作。我们可以看到msg.sender始终存在于合约运行环境中，无需引入或者申明；另外mapping的读取和写入也如同数组一样，通过键值对的方式写入和读取。

.. admonition:: 小练习

   请修改下列_createCar函数，让其除了发出事件以外，更能够记录下汽车Car和主人的两种对应关系：carToOwner和ownerCarCount：

   .. code-block:: solidity

      Car[] public cars;
      
      mapping (uint => address) public carToOwner;
      mapping (address => uint) ownerCarCount;
      
      function _createCar(string _name, uint _color) private {
          uint id = cars.push(Car(_name, _color)) - 1;
          _______[id] = msg.sender; // 此处填充
          ownerCarCount[________]++;  // 此处填充
          emit NewCar(id, _name, _color);
      }


require还是assert?
--------------------------

有时候我们会进行一定的函数条件检查，来判定是否可以接下去进行函数执行。读者会联想到if-else语法来进行判断，但是某些场合下，我们要求进行更严肃的权限检查，或者条件满足检查。require和assert两个关键字就应运而生，两者都在条件不满足时可以终止程序的运行，但是有如下区别。

  - require条件检查语句如果不通过，则扣除运行到当前语句时，程序执行所花费的 gas，终止程序执行，并返回。
  - assert 条件检查语句如果不通过，则视为严重错误，扣除所有的gas，终止程序执行，并返回.

例如以下程序将会检查发送方的字符串是否符合一定标准。

.. code-block:: solidity

   function sayHi (string _name) public returns (string) {
     require(keccak256(_name) == keccak256("Hello"));
     //条件满足，则执行:
     return "Hi!";
   }

这里位置上替换为assert关键字也是完全可行的。两者都会检查输入值是否是Hello。因为没有原生态的string比较函数，所以我们采用哈希的方法比较了两者的哈希值。Assert关键字相比于require更加具有惩罚性，经常用在检查变量范围上下溢出等场合，如果检查出错，表明程序出现了严重错误。而require则一般用在权限检查场合，检查是否有权操作合约等，权限不够则弹出提示，相对比较温和。

.. admonition:: 小练习

   我们不希望每个客户都创建无数的车。他们在我们合约内有且只能保留一辆车。所以创建第二辆车是不可能的。请改造如下函数，并仅允许合约调用者在无车的时候创建一辆：

   .. code-block:: solidity

      function createRandomCar(string _name) public {
         require(ownerCarCount[______] == ____); // 填充此处
         uint randColor = _generateRandomColor(_name);
          _createCar(_name, randColor);
      }


继承和引入
-------------------

智能合约的代码来源可以来源于自身项目内，也可以来源于外部已经早已部署完毕的链上合约。使用合约继承语法，不但可以减少重复的代码数量，也可以将代码更清晰地划分成数个组成部分。

.. code-block:: solidity

   contract Dog {
     function bark() public returns (string) {
       return "Wong!";
     }
   }
   
   contract BabyDog is Dog {
     function feed() public returns (string) {
       return "Drink some milk.";
     }
   }

这里小奶狗 BabyDog 继承了狗 Dog 的合约（通过 is 关键字），他们俩都具有bark()方法，同时 BabyDog还具有独特的feed()方法。

但是合约的代码不可能总是正好处在同一个文件内，我们经常要应用其他项目中的合约文件。怎么操作呢？我们可以将其分成两个文件，并放置在同一个目录下，并通过import 关键字来引入，还是用 Dog 合约来举例。

.. code-block:: solidity

   contract Dog {
     function bark() public returns (string) {
       return "Wong!";
     }
   }
   
   import "./Dog.sol"
   
   contract BabyDog is Dog {
     function feed() public returns (string) {
       return "Drink some milk.";
     }
   }

.. admonition:: 小练习

   请填充如下文件CarMaking.sol ，让合约能够顺利继承CarFactory。

   .. code-block:: solidity

      pragma solidity ^_________;
      _______ "./CarFactory.sol";
      
      contract CarMaking is CarFactory {
          
      }

省钱妙招：内存变量
---------------------------

在以太坊虚拟机讲解的时候，我们提到了不同的存储类型，花费的gas数额不同。它们的最终存储地方也不同。有时候为了省钱，我们会把临时变量留在内存里，而不是保存在区块链上。随着程序执行，内存里的变量会消亡，而区块链上的会永存。由于没有改变区块链状态，内存变量(memory)的花费会比状态变量(storage)的花费少很多。

.. code-block:: solidity

   contract Restaurant {
     struct Hamburger {
       string name;
       string status;
     }
   
     Hamburger[] hamburgers;
   
     function eatHamburger(uint _index) public {
       
       // Hamburger myHamburger = hamburgers[_index];
       // 上面这句编译器给一个 warning，然如果我们用下列代码，则warning消失
       Hamburger storage myHamburger = hamburgers[_index]; // storage 关键字
       // 直接修改了区块链上的数据
       myHamburger.status = "Eaten!";
       
       // 也可以使用 memory 关键字
       Hamburger memory anotherHamburger = hamburgers[_index + 1];
       // 此时修改的是内存中的数据，区块链不收影响
       anotherHamburger.status = "Eaten!";
       // 强制回写，影响区块链上的数据
       hamburgers[_index + 1] = anotherHamburger;
     }
   }

上述分别使用了 storage 和 memory 关键字来区别我们索引的对象，可以看见当我们用 storage 显式声明了之后，指针 myHamburger 指向了区块链上的某一个存储类型的数据，修改myHamburger后，立即在区块链上生效。而memory关键字神明的anotherHamburger 则不然，它仅为一份存储类型数据的内存拷贝，任何修改都不影响原数据，仅在内存中生效，如果想让修改在区块链上生效，必须回写到存储类型的数据上。


接口与合约调用
------------------------

合约的接口就是合约的抽象。我们可以通过定义合约接口，并指定合约地址，来调用另外一个在以太坊上早已经部署好的合约。例如下的合约。

.. code-block:: solidity

   contract MyNumber {
     mapping(address => uint) numbers;
   
     function setNum(uint _num) public {
       numbers[msg.sender] = _num;
     }
   
     function getNum(address _myAddress) public view returns (uint) {
       return numbers[_myAddress];
     }
   }

这个合约可以提炼成为一个简单的合约接口：

.. code-block:: solidity

   contract NumberInterface {
     function getNum(address _myAddress) public view returns (uint);
   }

我们因为只关心getNum函数来获取数字，所以就定义了getNum这一个合约接口函数。那么合约如何使用呢？我们可以配合合约地址来使用，如下所示。

.. code-block:: solidity

   contract MyContract {
     //取得已经部署好的合约的地址
     address NumberInterfaceAddress = 0x1E24F805d89211eD515dD8A4A8C54f96a3E0C1FE
     // 初始化合约，获得合约实例
     NumberInterface numberContract = NumberInterface(NumberInterfaceAddress);
     function someFunction() public {
     //调用合约的方法
       uint num = numberContract.getNum(msg.sender);
    }
   }


.. admonition:: 小练习

   请为如下的合约生成接口，命名该接口，并调用该接口的方法。
   
   .. code-block:: solidity

      contract Dog {
        function bark() public returns (string) {
          return "Wong!";
        }
      }
      
      contract DogInterface {
        function ____() ______ _______ (______);
      }
      
      contract MyContract {
        //取得已经部署好的合约的地址
        address DogInterfaceAddress = 0x735E388e9A8a073f14bdbb1C2bd4704dd386213c
        // 初始化合约，获得合约实例
        DogInterface dogContract = ____________(____________);
        function someFunction() public {
         //调用合约的方法
         string message = dogContract._____();
       }
      }

多返回值
-----------------

Solidity 的语法对于返回值并没有强制规定是一个单值，相反它鼓励多值返回来减少编程复杂度。多值返回的语法相对简单，如下所示。

.. code-block:: solidity

   // 申明要返回3个值
   function someFunction() internal returns(uint a, uint b, uint c) {
     return (1, 2, 3);  //封装，返回3个值
   }
   
   function processMultipleReturns() external {
     uint a;
     uint b;
     uint c;
     //多值返回，直接解封装:
     (a, b, c) = someFunction();
   }
   
   function getLastReturnValue() external {
     uint c;
     //我们也可以直接抛弃某些不关心的值:
     (,,c) = someFunction();
   }
