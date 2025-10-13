:github_url: https://github.com/laalaguer/ethereum-compass/blob/master/ch8/basic.rst

基础概念
================

本小节将重点介绍Solidity语法的基础，从最基本的合约结构和函数变量讲起。知识点都不复杂，读者只要带着纸笔填写练习就好，为后面的章节学习打下良好的基础。

没有浮点数运算
-----------------------

以太坊虚拟机给程序员的第一个惊(jing)喜(xia)就是： **它不会浮点数运算** ！诸如 1+2=3 的运算它是绰绰有余的，你也可以放入 2*100=200 做乘法运算。

但很不幸的是，没有一个变量可以表示 ``3.14`` 这样的数字，因为虚拟机根本不支持浮点数运算。

为了弥补这个缺陷，如何表示带小数的数字呢？以太坊将自身的数切分为最小 ``18`` 位，我们称为 ``wei`` ，任何数字都是wei的整数倍。具体的换算表如下。

+---------------------+----------+---------------------------+
| 单位                | wei 值   | wei                       |
+---------------------+----------+---------------------------+
| wei                 | 1 wei    | 1                         |
+---------------------+----------+---------------------------+
| kwei (babbage)      | 1e3 wei  | 1,000                     |
+---------------------+----------+---------------------------+
| mwei (lovelace)     | 1e6 wei  | 1,000,000                 |
+---------------------+----------+---------------------------+
| gwei (shannon)      | 1e9 wei  | 1,000,000,000             |
+---------------------+----------+---------------------------+
| microether (szabo)  | 1e12 wei | 1,000,000,000,000         |
+---------------------+----------+---------------------------+
| milliether (finney) | 1e15 wei | 1,000,000,000,000,000     |
+---------------------+----------+---------------------------+
| ether               | 1e18 wei | 1,000,000,000,000,000,000 |
+---------------------+----------+---------------------------+

请不要担心，上表看上去这么复杂，wei、kwei、mwei、gwei 算比例让人头晕，其实最重要的仅有最后一条，

.. centered:: :math:`1 {ether} = 10^{18} {wei}`

也就是说，当我们说1以太币的时候，在运行智能合约的虚拟机眼中，是相当于 1 x :math:`10^{18}` 的一个数。平时我们会用库函数来辅助我们构建数字，不用担心漏数了 ``0`` 的情况。

.. admonition:: 小练习

   请填写以下的等式，让 wei 和 ether 对应起来：

   .. code-block:: bash

      1 ether = ________________ wei

      314.159 ether = ____________ wei 

      0.156 ether = ____________ wei


合约基础
----------------

所有智能合约的代码都是包含在一组 ``{}`` 包含的 ``contract`` 关键字里的，没有例外。

合约是以太坊合约的基本”砖块”：所有的函数和变量都包含在这组花括号内，所有程序都是合约砖块的堆砌和组织。例如下方的 Hello 合约。

.. code-block:: solidity

   contract Hello {

   } 

预编译指令 ``pragma`` 也是合约的基础，它比较特殊，每份合约文件申明在开头，颇有 C/C++ 的风范。

pragma 指定了该使用哪个以太坊字节码编译器来编译这份源代码。
以太坊的编译器迭代了多个版本，指定一个没有bug的编译器也是一种技巧保障安全。

预编译指令非常简单，包含了想采用的编译器的版本，例如下方代码指定了 ``0.4.24`` 版本。

.. code-block:: solidity

   pragma solidity ^0.4.24;

.. admonition:: 小练习

   请填充下方的空白处，创建一份智能合约，将其命名为汽车工厂CarFactory， 指定编译器版本为0.4.24：

   .. code-block:: solidity
   
      pragma solidity ^_____;

      contract ______ {
      
      }

变量类型
-------------------

智能合约的变量分为两种，存在区块链上的和不存在区块链上的。存在区块链上的我们称为状态变量（state variable）。这类变量将永久记录在区块链上，写入读取它们就仿佛操作一个数据库一样，修改和赋值都会造成巨额的开销。不存在于区块链上的变量则是程序中的内存变量（memory variable），程序运行完毕就从内存中释放，相对开销较小。

例如将一个值100复制给一个状态变量 amount。

.. code-block:: solidity

   contract Example {
       uint amount = 100; //永久记录于区块链上
   }

``uint`` 变量类型即为unsigned integer的缩写，学过C/C++的同学一定感到很熟悉，该类型存储了一个非负的整数。实际上Solidity里面有多种位长的 uint 可以供我们选择，例如 ``uint8、uint32、uint256`` 等，实际使用中 ``uinit256`` 和 ``uint`` 是等价语法。

256位足够存储我们上述所提到的 10^18 wei 的空间还绰绰有余。

在本教程大部分代码里，将不区别 uint256 和 uint；在能用 uint 的地方尽量使用 uint 来保持数字的容量足够大。

.. admonition:: 小练习

   请在下面的空白处填写，让colorDigits 等于16，我们日后用它记录汽车颜色:

   .. code-block:: solidity

      pragma solidity ^0.4.24;
      
      contract CarFactory {
          //这里填写
          uint colorDigits = ____;
      }


运算符号
--------------------

Solidity 里面的数学运算都是普通的数学运算。加减乘除都与惯常理解一致。唯一的特殊点在于指数运算，例如10^18表示为如下的形式。

.. code-block:: solidity
   
   uint x = 10**18

其余的算术运算如下。

.. code-block:: solidity

   uint a = 3 + 5; // 8
   uint b = a - 2; // 6
   uint c = b * 5; // 30
   uint d = c % 7 // 7 * 4 = 28, 余数为 2

.. admonition:: 小练习

   请在下面空白处填写，让 colorModulus 等于10的 colorDigits次方，这样我们每次做除法的时候，可以保证余数的不超过colorDigits 位。

   .. code-block:: solidity

      pragma solidity ^0.4.24;
      
      contract CarFactory {
      
          uint colorDigits = 16;
          //这里填写
          uint colorModulus = ____ ** ___;
       
      }


结构体 Struct
------------------

有时候 Solidity 提供的基本数字类型、文字类型并不能封装我们需要的数据结构，在面向对象的编程语言中，由于函数返回只能返回一个值，所以对返回结果进行了大量的封装、解封装的操作。在Solidity中我们也可以封装数个基本类型为一个结构体，例如我们面对一个“人”对象的时候，可以将他的年龄和姓名封装入一个结构体中。

.. code-block:: solidity

   struct Person {
     uint16 age; //16位应该能涵盖大部分正常人类寿命
     string name; // 例如 name = “Peter Wilson Jr.”
   }

上述结构体struct是Solidity语言中预置的关键字，帮助我们将数个基本类型进行封装成一个通用的结构体。结构体struct在高级用法中并不只是封装了数据这么简单，它还能作为编译器优化的手段来节约代码运行、存储时候的gas花费。
这里我们介绍一个新类型 ``string`` -- **任意长度** 的字符串，每个字符是 ``utf-8`` 类型的值。

.. admonition:: 小练习

   请在空白处填写，创建一个汽车结构体Car，Car拥有一个名称name和颜色color属性：
   
   .. code-block:: solidity

      struct _____ {
          string ___;
          uint ___;
      }


数组array
-------------------

当我们想创建同类型数据的集合的时候就用上了 array 数组。一个数组里面可以加入同类型的数据，哪怕是 struct 类型的数据都可以是数组的基本类型。例如下面的数组。

.. code-block:: solidity

   uint[3] fixed; // 定长数组，只能包含3个元素，每个元素是 uint
   string[10] stringArray; // 定长数组，只能包含10个元素， 每个元素是 string
   Person[] dynamic; // 可变长数组，可持续增长，每个元素是 Person 结构体

可变长数组给我们提供了一个机会，类似数据库，可以持续往这个“篮子”里面写入和读取数字，在写入读取的时候势必会有权限问题(可不可以被合约外访问到？)，我们采用 public关键字来修饰变量，让变量可以公开被合约外访问到，但该访问并不包含修改权限，仅仅包含了读取权限。

.. code-block:: solidity

   Bike[] public bikes;

.. admonition:: 小练习

   请创建一个可容纳Car类型的动态长度数组cars：
   
   .. code-block:: solidity

      pragma solidity ^0.4.24;
      
      contract CarFactory {
      
          uint colorDigits = 16;
          uint colorModulus = 10 ** colorDigits;
      
          struct Car {
              string name;
              uint color;
          }
      
      // 这里填充
      ____[] public ____;
      }

函数申明
------------------

一个智能合约的“能动”部分就是函数。函数承担了数据读取，数据修改，以及数据存储的触发。

智能合约并没有一个入口main函数来执行整个程序。

你可以把它类比为Web后端开发中为响应请求而写的一个一个 Request Handler，也可以理解为 Android 编程中为响应外部生命周期调用而存在的各个响应函数。

只要记住一个中心思想：智能合约的函数调用是“被动的”，需要外部主动来触发。

我们很容易构建一个函数，指定它的输入参数。

.. code-block:: solidity
   
   function drinkTea(string _name, uint _amount) {
   }  

   drinkTea("Lemon Tea", 100);

上方我们申明了一个 ``drinkTea`` 函数，接收两个参数(一般函数参数用下划线_开头以区别于全局变量)，在调用时候采用数值直接填写方式调用即可。

.. admonition:: 小练习

   请在下方创建函数createCar, 并且该函数接受两个参数，_name (string 类型)和 _color( uint类型):

   .. code-block:: solidity

      function ________ (_____ _name, _____ _color) {
      
      }

函数有了，我们接下来填充这个函数的代码，让它能够执行一定的任务。例如生成一些数据并且填充。在 Solidity 中，数据可以被组织进入数组 Array 中，而数据类型可以随意选择。例如我们之前提到的 Person 数据结构，我们基于它构建一个people数组。

.. code-block:: solidity

   struct Person {
     uint16 age;
     string name;
   }

   Person[] public people;

我们可以申明新的 Person 并且加入 people 里面，不断扩充这数组，例如我们创建Steve Jobs这人物并且填充入数组。

.. code-block:: solidity

   Person steve = Person(56, "Steve Jobs"); // 申明该人物
   people.push(steve); // 填充进入数组
   people.push(Person(56, "Steve Jobs"));// 也可以简化为一行代码更紧凑

.. admonition:: 小练习

   请填充我们的createCar函数，并且创建一个Car结构体加入已有的Car数组内。

   .. code-block:: solidity

      Car[] public cars;
      function createCar(string _name, uint _color) {
         cars.push(_____ (_____, ______));
      }

很好，函数的介绍部分基本完成了。目前为止我们尚未接触到函数权限问题。作为语言间的对比，在Java中公开/私有函数都有 public/private等权限修饰，在Python/JavaScript中则没有私有函数，全部是公开函数，全靠程序员自觉的编程习惯。Solidity中默认的函数权限是Public，也就是完全公开。有时候这是不可取的。我们可以用 private 来修饰这些函数，例如下方的代码所示。

.. code-block:: solidity

   uint[] digits;
   
   function _addToArray(uint _number) private {//修饰符在最后
     digits.push(_number);
   }

这样该函数仅在本合约内可以被调用，并不会被外界感知或者调用到。我们通常约定俗成地将private修饰的函数名字前缀加上下划线 _ 来提醒程序员这里是私有函数。

.. admonition:: 小练习

   请将我们的下属函数修改为private修饰的函数，注意createCar已经有下划线前缀：

   .. code-block:: solidity

      Car[] public cars;
      function _createCar(string _name, uint _color) _______ {
         cars.push(Car(_name, _color));
      }

除了函数 private/public 修饰符以外，还有相应的 internal/external 修饰符，internal 修饰符可以让合约继承后子合约访问该函数；external 修饰符让该函数只能被外部调用者调用。

一个有用的函数，还应该将处理结果返回给调用者，例如下方的函数返回一个Person类型的返回值。

.. code-block:: solidity

   string name = "John";
   
   function makePerson (uint16 _age) public returns (Person) {//注意使用了小括号
     return Person(_age, name);
   }

这里Person 两边使用了小括号，这点尤其有意思。其实返回值也可以是两个或以上的值，这和Python的语法相似，灵活性较Java/JavaScript有所提升，我们后文会提及。
Solidity的函数也有修饰符，称为modifiers，这标明了函数可能对区块链状态有无修改/读取的标记。一般都会标记该值让编译器帮我们执行代码的优化。例如下面两段代码。

.. code-block:: solidity

   Person[] public people;
   function viewMe() public view returns (Person) {
       return people[0];// 读取了区块链数据区的people
   }
   
   function _multiply(uint a, uint b) private pure returns (uint) {
     return a * b; // 未读取任何区块链数据， 单纯的计算
   }


这里我们看见了两个修饰符，一个是view标明单纯的“查勘”类型的函数，它会读取记录在区块链上的数据，但它并不修改数据，是个只读操作。一个是pure标明是一个纯粹的函数，它和区块链上的数据无关，仅仅进行某种内存中的运算而已。那么读者会问，不标记任何修饰符的函数呢？那通常默认就是对区块数据会进行写操作的函数了。

.. admonition:: 小练习

   请创建一个函数 _generateRandomColor，并且该函数是私有的，仅读取区块链数据的，并且返回 uint 类型的值作为返回值。

   .. code-block:: solidity

      function _generateRandomColor (string _str) _____ _____ returns (_____) {
      
      }

   请创建一个函数 divideNumbers，并且该函数是公开的，不读取/修改区块链数据的，并且返回 uint 类型的值作为返回值。

   .. code-block:: solidity

      function divideNumbers(uint _a, uint _b) _____ _____ returns (_____) {
      
      }

类型转换与内置函数
----------------------------

和面向对象的语言一样，Solidity包含了类型转换，它并不会帮你进行向下的类型转换操作，例如如下的操作会导致错误。

.. code-block:: solidity

   uint8 a = 10;
   uint b = 20;
   // uint(uint256)类型太大了，无法塞入 uint8保存
   uint8 c = a * b;
   // 但是强制类型转换后就可以了
   uint8 c = a * uint8(b);

在 Solidity 编程中有数个常用函数是内置送给开发者使用的，就和Python/JavaScript 中环境自带的函数一样，其中一个函数经常用到，就是keccak256哈希函数，这是我们前文经常提到的一个散列函数算法，可以根据任意长度的明文产生固定长度256位的哈希值。256位又正好和 uint 的位数相对应。例如：

.. code-block:: solidity

   keccak256("aaaab");
   //6e91ec6b618bb462a4a6ee5aa2cb0e9cf30f7a052bb467b0ba58b8748c00d2e5


有了这工具，我们可以轻松地根据输入来生成一个“伪随机”的256位值。这里 keccack256函数并不是一个很好的随机源，因为它对固定输入产生的输出相同。好的随机源一般包含了操作系统里的噪音。在区块链上产生一个随机数是困难的，因为区块链每一步都讲究可验证，那么每次运行程序的结果应该相同：这意味着每次运行随机函数的输出亦应该一致。

.. admonition:: 小练习

   请将下列函数填充完整，输入_str 后该函数将会填充如 keccack 函数进行哈希，并生成一个256位的哈希值，请强制转换它为256位的 uint 并和 colorModulus 作取余操作。

   .. code-block:: solidity

      function _generateRandomColor(string _str) private view returns (uint) {
          uint rand = ____(keccak256(_str));
          return rand % ____;
      }

   请再创建一个函数 createRandomCar 该函数将接受一个车名 _name 作为输入值。该函数没有任何返回值，并且调用_generateRandomColor 产生汽车的颜色。之后调用 _createCar 函数（前文已经提及）将新产生的车子推入区块链数据中永久存储。

   .. code-block:: solidity

      function __________(string _name) public {
          uint randColor = _________(_name);
          _______(_name, randColor);
      }

合约与事件
--------------------


在前述以太坊虚拟机章节，我们讲过虚拟机的输出仅仅包含了两种手段：修改合约的区块链数据区域，或者产生日志输出。日志输出的内容组成部分就包含了“事件”。熟悉编程的读者肯定知道，日志产生后可以经常被其他程序读取，作为事后分析，或者某状态快照的参考信息。例如以太坊上对某些智能合约数据修改后，往往会主动产生日志记录下来，方便日后查询。下方的emit关键字代表了一次日志的产生。

.. code-block:: solidity

   event PersonCreated(string name, uint16 age);
   
   function create(string _name, uint16 _age){
       emit PersonCreated(_name, _age);// 直接产生了日志
       return Person(_name, age);
   }

这里值得注意的是，日志的产生一定要用emit关键字，这在新版的语法里面是强调的。虚拟机收集了日志之后会妥善存储，并不用编程人员操心日志的去处。日志的收集往往会被前端调用该合约的程序所捕获，并且相应地展示出来UI结果。现在市面上的以太坊轻钱包都是根据日志整理出用户的各种代币余额的，日志极大地简化了前端开发中“遍历”区块链的负担。

.. admonition:: 小练习

   我们的CarFactory.sol合约接近完成了，请填充下面空白处，让合约能够产生NewCar事件，并且改造_createCar 函数让每辆车进入区块链数据区保存后能产生事件，通知外界区块链数据的变化。

   .. code-block:: solidity

      pragma solidity ^0.4.24;
      
      contract CarFactory {
      
          event _______(uint carId, string name, uint color);// 填充此处
      
          uint colorDigits = 16;
          uint colorModulus = 10 ** colorDigits;
      
          struct Car {
              string name;
              uint color;
          }
      
          Color[] public cars;
      
          function _createCar(string _name, uint _color) private {
              uint id = cars.push(Car(_name, _color)) - 1;
              emit _______(id, _name, _dna); // 填充此处
          } 
      
          function _generateRandomColor(string _str) private view returns (uint) {
              uint rand = uint(keccak256(_str));
              return rand % colorModulus;
          }
      
          function createRandomCar(string _name) public {
              uint randColor = _generateRandomColor(_name);
              _createCar(_name, randColor);
          }
      }


