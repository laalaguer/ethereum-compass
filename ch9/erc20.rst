上手实践：ERC20合约
==============================

让我们暂时把MetaCoin项目放一边。我们需要从头开始建立一个自己的合约项目。较为合适举例的就是ERC20数字资产合约。作为2017-2018年以太坊上最火热的Dapp应用之一，ERC20数字资产合约承载了数千项目的早期融币流通的功能，同时也因为简洁优雅的接口设计让其迅速成为事实上的标准并被交易所、钱包程序所认可。时至今日，世界各地的项目在早期启动的时候之所以考虑以太坊，也是考虑到在以太坊上建立ERC2标准数字资产的编程便利性

我们在本节从头构建一份ERC20的合约代码，在项目过程中加入编译技巧与测试的技巧，向读者展示一份承载数万人数字资产的合约是如何从0起步走向生产环境的。

新建项目目录
--------------------

我们不再下载任何的Truffle Box, 建立一个新的空白项目即可。

.. code-block:: bash

   $ mkdir erc20-test
   $ cd erc20-test/
   $ truffle init
   
至此，我们的 ERC20项目已经有了一个基本框架，我们查看一下目录结构。

.. code-block:: bash

   erc20-test/
   ├── contracts
   │   └── Migrations.sol
   ├── migrations
   │   └── 1_initial_migration.js
   ├── test
   ├── truffle-config.js
   └── truffle.js

因为要使用本地的Ganache/Test-RPC的测试客户端，我们修改truffle.js加入测试网络的信息，测试网络将运行在本地，且监听在8545端口。以此为基础，接下去我们将开始 ERC20的相关合约开发。

.. code-block:: javascript

   module.exports = {
     networks: {
         development: {
             host: "localhost",
             port: 8545,
             network_id: "*" // Match any network id
         }
     }
   };


ERC20 Basic合约接口
--------------------------------

最早的 ERC20合约仅支持部分函数与公开属性，它受到社区改进提议EIP179所提出的标准启发，共支持3个函数和一个事件，它的代码如下。

.. code-block:: solidity

   pragma solidity ^0.4.24;
   
   /**
    * @title ERC20Basic
    * @dev Simpler version of ERC20 interface.
    * See https://github.com/ethereum/EIPs/issues/179
    */
   contract ERC20Basic {
       // Total supply of token.
       function totalSupply() public view returns (uint256);
       // Balance of a holder _who
       function balanceOf(address _who) public view returns (uint256);
       // Transfer _value from msg.sender to receiver _to.
       function transfer(address _to, uint256 _value) public returns (bool);
       // Fired when a transfer is made
       event Transfer(
           address indexed from, 
           address indexed to, 
           uint256 value
       );
   }

其中各个代码和函数接口解释如下。
  - totalSupply()函数：公开查询合约发行Token总量，由于需要查询区块链数据，故采用public和view来修饰该方法。返回值是最大宽度的256bit的正整数。
  - balanceOf()函数：公开查询合约中某地址所持有的Token总量，接收一个地址参数，查询权限公开，任何人都可以查询任何其他人的Token总量。返回值也是最大宽度的256bit的正整数。
  - transfer()函数：直接转账Token的函数。由转账发起方负责呼叫此函数。函数接受两个参数，地址参数 _to 和转账数量 _value，不用与以太坊的交易体的value 混淆，这里的 _value 特指Token的数量，由256bit的正整数指定。这里发送方是暗含在以太坊虚拟机执行的上下文中的，为msg.sender意即函数调用方。函数返回值是个布尔值，如遇余额不足等情况转让Token失败，函数返回False。

这个合约接口的定义中还包含了Transfer事件，记录的事件为发送方、接收方和转让 Token 的数额。值得注意的是在安全性要求下transfer()函数必定要检查发送方的余额是否足够转账，否则会引发任意转账漏洞，造成Token被盗。

ERC20 合约接口
----------------------

ERC20合约在ERC20 Basic合约上进行了部分扩容，增加了函数定义.转让Token的过程可以由“主动转账”变为“授权索取”，在便利性上而言,主动转账更为直接，但要求知道转让对象是谁；而被动索取更适用于家长-孩子关系中管理的零花钱模式，让家长能够定授权孩子动用一部分的资金，至于资金流向何方，是孩子的决定权。ERC20代码如下。

.. code-block:: solidity

   pragma solidity ^0.4.24;
   
   import "./ERC20Basic.sol";
   /**
    * @title ERC20 interface
    * @dev Enhanced interface with allowance functions.
    * See https://github.com/ethereum/EIPs/issues/20
    */
   contract ERC20 is ERC20Basic {
       // Check the allowed value that the _owner allows the _spender to take from his balance.
       function allowance(address _owner, address _spender) public view returns (uint256);
   
       // Transfer _value from the balance of holder _from to the receiver _to.
       function transferFrom(address _from, address _to, uint256 _value) public returns (bool);
   
       // Approve _spender to take some _value from the balance of msg.sender.
       function approve(address _spender, uint256 _value) public returns (bool);
       
       // Fired when an approval is made.
       event Approval(
           address indexed owner,
           address indexed spender,
           uint256 value
       );
   }

我们可以清晰地看到代码中有一层继承，就是 *“contract ERC20 is ERC20Basic”* 这句话。合约接口依然只是定义，并没有具体的实现方式，它的接口定义如下。
  - allowance()函数：查阅授权情况。这是个公开函数，任何人都可以查询任何其他人的授权情况，函数接受两个参数，参数第一个是授权人 _owner，第二个是被授权人 _spender。因为查询了区块链相关的存储区，所以用public和view来修饰该函数，函数返回值是授权token的数量，采用最宽位256bit正整数来表示。
  - approve()函数：允许授权行为，持有者允许被授权人转走一定数量的Token资产。这是个公开可调用函数，但修改了区块链状态，故仅采用public进行修饰。函数接收两个参数，第一个是 _spender被授权人，第二个是 _value即授权的 Token数量。这个函数有一定的问题，在被授权人花掉token的时候若授权方调整了数额，则有一定概率会发生授权过多的现象。该函数执行前提是检查msg.sender是否有足够的额度可供授权。
  - transferFrom()函数：被授权人划走一定量的Token去往他指定的地点。这个函数公开可调用，但会修改区块链状态。在划走之前一定要检查权限，是否该人被授权动用了这些额度的Token。函数共接收三个参数 _from、_to、 _value。分别代表了转移支付方，转移受付方，以及转移Token的额度。

该合约还定义了Approval 事件，该事件与Transfer事件一样，一旦发生相应的行为就会被触发，Approval事件记录了授权事件的授权方、被授权方和授权的数额。


SafeMath基础数学库
---------------------------

以太坊上因为没有整型边界检查，所以号称最安全的合约语言其实有很大几率会整形溢出，也就是在最大值上再+1，就上溢出，变成最小值；最小值-1也向下溢出，变为最大值。这样会无端造成账户财产增多或者减少，造成合约的参与者的损失。

在我们的ERC20合约里面必定会用上加减两样算术。以太坊上的整型溢出问题我们必须格外小心处理。我们可以小心翼翼处理每一个加减法的地方，也可以直接用 openZepplin 的合约库SafeMath来帮我们处理加减法。我们截取一段如下。

.. code-block:: solidity

   pragma solidity ^0.4.24;
   
   /**
    * @title SafeMath
    * @dev Math operations with safety checks that throw on error
    */
   library SafeMath {
       /**
       * @dev Multiplies two numbers, throws on overflow.
       */
       function mul(uint256 _a, uint256 _b) internal pure returns (uint256 c) {
         // Gas optimization: this is cheaper than asserting 'a' not being zero, but the
         // benefit is lost if 'b' is also tested.
         // See: https://github.com/OpenZeppelin/openzeppelin-solidity/pull/522
           if (_a == 0) {
               return 0;
           }
   
           c = _a * _b;
           assert(c / _a == _b);
           return c;
       }
   
       /**
       * @dev Integer division of two numbers, truncating the quotient.
       */
       function div(uint256 _a, uint256 _b) internal pure returns (uint256) {
           // assert(_b > 0); // Solidity automatically throws when dividing by 0
           // uint256 c = _a / _b;
           // assert(_a == _b * c + _a % _b); // There is no case in which this doesn't hold
           return _a / _b;
       }
   
       /**
       * @dev Subtracts two numbers, throws on overflow (i.e. if subtrahend is greater than minuend).
       */
       function sub(uint256 _a, uint256 _b) internal pure returns (uint256) {
           assert(_b <= _a);
           return _a - _b;
       }
   
       /**
       * @dev Adds two numbers, throws on overflow.
       */
       function add(uint256 _a, uint256 _b) internal pure returns (uint256 c) {
           c = _a + _b;
           assert(c >= _a);
           return c;
       }
   }

以上的合约库SafeMath的基本逻辑就是。
  - 乘法mul()：检查两个乘数，是否由一方为0，如果都不为零，则乘法结果处以其中一个乘数，应该等于另一个乘数。如果不是，则发生了上溢出现象。
  - 除法div()：较为简单，如果除数为零虚拟机直接报错。
  - 减法sub()：确保减数永远小于被减数，否则表示减法发生了下溢出现象。
  - 加法add()：确保加和后的结果值大于两个相加因子，否则发生了上溢出现象。

值得注意的是，SafeMath库合约的写作过程中每个function方法都自带internal pure修饰，表明这些方法都可以继承，且都不修改或读取任何区块链的数据，是工具方法。下面我们组合这些方法发行一个猫币数字资产，代号CAT。


猫币：CAT数字资产合约
--------------------------

“万事俱备，只欠东风。”我们的猫币数字资产合约马上就可以上市流通啦！

经过上述的合约接口与函数库分析，我们可以看到一条合约继承的链条，我们要编写CAT猫币数字资产合约的话，必须继承自ERC20接口并实现其中的所有方法，为了安全，我们对所有输入的变量作SafeMath运算保障安全。合约继承关系如图 9-2_ 所示。

.. _9-2:
.. figure:: /img/Picture54.png
   :align: center
   :width: 600 px

   数字资产合约 CAT的合约继承关系

此时项目目录合约部分添加完毕，结构如下所示。

.. code-block:: bash

   erc20-test/
   ├── contracts
   │   ├── Cat.sol
   │   ├── ERC20.sol
   │   ├── ERC20Basic.sol
   │   ├── Migrations.sol
   │   └── SafeMath.sol
   ├── migrations
   │   └── 1_initial_migration.js
   ├── test
   ├── truffle-config.js
   └── truffle.js

Cat.sol合约代码我们分拆开来解析，分析如下。

.. code-block:: solidity

   pragma solidity ^0.4.24;
   
   import "./ERC20.sol";
   import "./SafeMath.sol";
   
   /**
    * @title CAT Token
    * @dev Compatible with ERC20/VIP180 Standard.
    * Special thanks go to openzeppelin-solidity project.
    */
   contract CAT is ERC20 {
       using SafeMath for uint256;
   
       // Name of token
       string public constant name = "CAT Token";
       // Symbol of token
       string public constant symbol = "CAT";
       // Decimals of token
       uint8 public constant decimals = 18;
       // Total supply of the tokens
       uint256 internal totalSupply_;
   
       // balances: (_holder => _value)
       mapping(address => uint256) public balances;
   
       // allowed: (_owner, => (_spender, _value))
       mapping (address => mapping (address => uint256)) internal allowed;


合约文件开头立即引用了SafeMath.sol和ERC20.sol两个库。之后定义了合约名称CAT,继承自ERC20标准。为简略引用SafeMath，每个 uint256的地方使用加减乘除函数，申明用SafeMath来操作两个或以上的操作数。

.. centered:: using SafeMath for uint256;

在这之后，代码片段又定义了合约的变量和常量。
  - name: 数字资产名称，CAT Token；public修饰的变量自动生成getter方法。
  - symbol: 数字资产代号，CAT；public修饰 的变量自动生成getter方法。
  - decimals: 数字资产小数点位数，我们选择最大位数18位。
  - totalSupply\_ ：内部使用变量，不可外泄，发型Token总量。非公开变量，故采用 internal 修饰符修饰。
  - balances: 持有Token的账户地址与Token余额的映射记录，可公开查询。
  - allowed: 授权人、被授权人、授权数量的映射。需由函数进行操作，非公开变量，用internal 修饰符修饰。

.. code-block:: solidity

   constructor() public {
     totalSupply_ = 1 * (10 ** 10) * (10 ** 18); // 10 000 000 000 tokens of 18 decimals.
     balances[msg.sender] = totalSupply_;
     emit Transfer(0, msg.sender, totalSupply_);  // init mint of coins complete.
   }

以上代码表明我们共发行100亿枚 CAT币，每个币可以分为1018位小数。在创世的构造函数里，我们将所有的CAT币全数转给msg.sender也就是开创合约的人。接下去由他负责发送给任意想要获取CAT币的人。

.. code-block:: solidity

   // Get the total supply of the coins
   function totalSupply() public view returns (uint256) {
     return totalSupply_;
   }
   
   // Get the balance of _owner
   function balanceOf(address _owner) public view returns (uint256 balance) {
     return balances[_owner];
   }
   
   /** Make a Transfer.
   * @dev This operation will deduct the msg.sender's balance.
   * @param _to address The address the funds go to.
   * @param _value uint256 The amount of funds.
   */
   function transfer(address _to, uint256 _value) public returns (bool) {
     require(_to != address(0), "Cannot send to all zero address.");
     require(_value <= balances[msg.sender], "msg.sender balance is not enough.");
   
     // SafeMath.sub will throw if there is not enough balance.
     balances[msg.sender] = balances[msg.sender].sub(_value);
     balances[_to] = balances[_to].add(_value);
     emit Transfer(msg.sender, _to, _value);
     return true;
   }

与接口描述一致，totalSupply()函数负责反馈总Token数量，balanceOf()函数可以公开查看任意人的任意时刻的Token余额，只需要在 balances 映射里面找到相应的键值对查询即可。

这里尤其注意的是transfer()函数的实现，它在开头引用了两次require()函数，先确保转移发起方拥有足够多的Token余额，又检查了发送接收方是否是全零地址（0x000…000的地址），一般全零地址用于部署合约，有时候开发人员搞错了也会填写这个值。在这里检测一下是非常有必要的。之后的函数体实现中又采用了SafeMath 的add()方法代替了“+”符号，sub()方法代替了“-”符号，对转账双方的余额进行了安全的数学加减。在转账完成顺利无误后，会触发事件Transfer，让以太坊虚拟机记录下来并留存成交易收据存根。

.. code-block:: solidity

   /**
      * @dev Check the allowed funds that _spender can take from _owner.
      * @param _owner address The address which owns the funds.
      * @param _spender address The address which will spend the funds.
      * @return A uint256 specifying the amount of tokens still available for the spender.
     */
     function allowance(address _owner, address _spender) public view returns (uint256) {
       return allowed[_owner][_spender];
     }
   
   /**
    * @dev Transfer tokens from one address to another
    * @param _from address The address which you want to send tokens from
    * @param _to address The address which you want to transfer to
    * @param _value uint256 the amount of tokens to be transferred
   */
   function transferFrom(address _from, address _to, uint256 _value) public returns (bool) {
       require(_value <= balances[_from], "_from doesnt have enough balance.");
       require(_value <= allowed[_from][msg.sender], "Allowance of msg.sender is not enough.");
       require(_to != address(0), "Cannot send to all zero address.");
   
       balances[_from] = balances[_from].sub(_value);
       balances[_to] = balances[_to].add(_value);
       allowed[_from][msg.sender] = allowed[_from][msg.sender].sub(_value);
       emit Transfer(_from, _to, _value);
       return true;
   }
   
   /**
    * @dev Approve the passed address to spend the specified amount of tokens on behalf of msg.sender.
    * @param _spender The address which will spend the funds.
    * @param _value The amount of tokens to be spent.
   */
   function approve(address _spender, uint256 _value) public returns (bool) {
       allowed[msg.sender][_spender] = _value;
       emit Approval(msg.sender, _spender, _value);
       return true;
   }
   }

合约授权代码和合约转账代码在原理上很相似，安全考量也近似。allowance()函数通过检查映射的方式找到授权人和被授权人的授权额度信息；transferFrom() 函数在编写时特意使用require()检查了三次，分别是授权人是否账户余额充足、转账额度是否超过了授权额度、接收地址是否为全零地址。在被授权人转账走以后，还会相应地减少授权额。转账函数校验了msg.sender 是否有足够的授权进行转账操作，但是授权approve()函数却不强制要求授权人需要余额大于授权数量。例如现在只持有10枚币，哪怕现在夸下海口授权100枚给与他人也没关系。因为授权值高于账户Token余额的话，代码会保证转账授权失败的。

智能合约编写完毕！接着我们开始进行项目的Truffle测试。



