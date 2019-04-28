# 发行隐私币

## 启动节点

隐私币发行前，推荐大家在开发网测试一下，因此本文就以开发网开始，`Alpha`/`Beta`/`Mainnet`正式网类似。

打开第一个终端，并启动一个带`console`的节点。为了省事，我编辑了一个脚本，直接解锁并开启挖矿（此方法仅限于用户开发网络，正式网不推荐）。

{% code-tabs %}
{% code-tabs-item title="console\_dev.sh" %}
```bash
export LD_LIBRARY_PATH=$GOPATH/src/github.com/sero-cash/go-czero-import/czero/lib
gero --dev --datadir /path/to/sero/datadir/dev --devpassword 'password' --mine --minerthreads 1 console
```
{% endcode-tabs-item %}
{% endcode-tabs %}

如果一切正常，会输出版本号等提示，并进入控制台。

```bash
Welcome to the Gero JavaScript console!

instance: Gero/v0.7.0-beta.r7-hotfix.1/darwin-amd64/go1.12
coinbase: 4CZWS8sPWqi2CLFVKANJsif9xSEYVBFG42ma84P4VNghWBrEQAVL9ATELwYJF4FPkSnsMyj9fVSJbmjfj4JbYJ2Y
at block: 2082 (Fri, 26 Apr 2019 14:36:50 CST)
 datadir: /path/to/sero/datadir/dev
 modules: admin:1.0 debug:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 sero:1.0 txpool:1.0 web3:1.0

>
```

控制台的这个终端，还会时不时的输出日志，一会儿会用到这些日志。

现在打开第二个终端，`attach`进这个节点，用于部署合约。

```text
export DYLD_LIBRARY_PATH=$GOPATH/src/github.com/sero-cash/go-czero-import/czero/lib
gero --datadir /path/to/sero/datadir/dev attach
```

同样的，成功后，会输出和上边类似的提示，然后进入控制台。

## 发行

### 编辑源码

打开网址[https://remix.web.sero.cash/\#optimize=true&version=builtin](https://remix.web.sero.cash/#optimize=true&version=builtin)，会自动出现示例代码，新建一个源码文件，然后输入`Solidity`源码，如下

{% code-tabs %}
{% code-tabs-item title="BORONGTS.sol" %}
```javascript
pragma solidity ^0.4.16;

import "browser/seroInterface.sol";

library SafeMath {
    function safeMul(uint256 a, uint256 b) pure internal returns (uint256) {
        uint256 c = a * b;
        assert(a == 0 || c / a == b);
        return c;
    }

    function safeDiv(uint256 a, uint256 b) pure internal returns (uint256) {
        assert(b > 0);
        uint256 c = a / b;
        assert(a == b * c + a % b);
        return c;
    }

    function safeSub(uint256 a, uint256 b) pure internal returns (uint256) {
        assert(b <= a);
        return a - b;
    }

    function safeAdd(uint256 a, uint256 b) pure internal returns (uint256) {
        uint256 c = a + b;
        assert(c>=a && c>=b);
        return c;
    }
}

contract HasOwner {
    address public owner;

    constructor() public {
        owner = msg.sender;
    }

    modifier onlyOwner {
        require(msg.sender == owner);
        _;
    }

    function transferOwnership(address newOwner) onlyOwner public {
        owner = newOwner;
    }
}

contract BORONGTS is SeroInterface, HasOwner {
    
    using SafeMath for uint256;
    // Public variables of the token
    uint8 constant DECIMALS = 18;
    uint256 public totalSupply;
    string public tokenSymbol = "BORONGTS";
    
    constructor(uint256 initSupply, string tokenName) public payable {
        totalSupply = initSupply *10** uint256(DECIMALS);
        require(sero_issueToken(totalSupply, tokenName));
        tokenSymbol = tokenName;
    }
    
    function getDecimal() public pure returns (uint8) {
        return DECIMALS;
    }
        
    function transfer(address _to, uint256 _value) public onlyOwner {
        require(sero_balanceOf(tokenSymbol) >= _value);
        require(sero_send_token(_to,tokenSymbol,_value));
    }
    
    function reclaimSero(address _to) external onlyOwner {
        _to.transfer(address(this).balance);
    }
    
    function totalSupply() public view returns (uint256) {
        return totalSupply;
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

其中，合约的名字叫做`BORONGTS`，币的名称也叫`BORONGTS`，18位，总发行量可以在构造函数里指定。

```text
contract BORONGTS is SeroInterface, HasOwner {
    
    using SafeMath for uint256;
    // Public variables of the token
    uint8 constant DECIMALS = 18;
    uint256 public totalSupply;
    string public tokenSymbol = "BORONGTS";
    
    constructor(uint256 initSupply, string tokenName) public payable {
        totalSupply = initSupply *10** uint256(DECIMALS);
        require(sero_issueToken(totalSupply, tokenName));
        tokenSymbol = tokenName;
    }
    
    // ...其他代码...
    
}
```

你也可以按照自己的喜好，起一些其他的名字，6个字母以上，可以带下划线或者数字。

### 执行命令

如果代码编译通过，右侧会出现文件中的各个合约的名字，如下图

![Remix](.gitbook/assets/image%20%284%29.png)

点击`Details`，下拉到`WEB3DEPLOY`的代码处，点击拷贝。

![WEB3DEPLOY](.gitbook/assets/image%20%287%29.png)

把拷贝来的命令，修改成自己想要的参数。

```text
var initSupply = 1024 ;
var tokenName = 'BORONGTS' ;
var borongtsContract = web3.sero.contract([{"constant":true,"inputs":[],"name":"totalSupply","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"getDecimal","outputs":[{"name":"","type":"uint8"}],"payable":false,"stateMutability":"pure","type":"function"},{"constant":false,"inputs":[{"name":"_to","type":"address"}],"name":"reclaimSero","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"tokenSymbol","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"owner","outputs":[{"name":"","type":"address"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[{"name":"x","type":"bytes32"}],"name":"bytes32ToString","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"pure","type":"function"},{"constant":false,"inputs":[{"name":"_to","type":"address"},{"name":"_value","type":"uint256"}],"name":"transfer","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"newOwner","type":"address"}],"name":"transferOwnership","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"inputs":[{"name":"initSupply","type":"uint256"},{"name":"tokenName","type":"string"}],"payable":true,"stateMutability":"payable","type":"constructor"}]);
var borongts = borongtsContract.new(
   initSupply,
   tokenName,
   {
     from: web3.sero.accounts[0], 
     data: '0x7f3be6bf24d822bcd6f6348f6f5a5c2d3108f04991ee63e80cde49a8c4746a0ef36000557fcf19eb4256453a4e30b6a06d651f1970c223fb6bd1826a28ed861f0e602db9b86001557f868bd6629e7c2e3d2ccf7b9968fad79b448e7a2bfb3ee20ed1acbc695c3c8b236002557f7c98e64bd943448b4e24ef8c2cdec7b8b1275970cfe10daf2a9bfa4b04dce9056003557fa6a366f1a72e1aef5d8d52ee240a476f619d15be7bc62d3df37496025b83459f6004557ff1964f6690a0536daa42e5c575091297d2479edcc96f721ad85b95358644d2766005557f9ab0d7c07029f006485cf3468ce7811aa8743b5a108599f6bec9367c50ac6aad6006557fa6cafc6282f61eff9032603a017e652f68410d3d3c69f0a3eeca8f181aec1d176007557f6800e94e36131c049eaeb631e4530829b0d3d20d5b637c8015a8dc9cedd70aed60089081557fbbf1aa2159b035802d0a4d44611849d5d4ada0329c81580477d5ec3e82f4f0a66009557fa8b83585a613dcf6c905ad7e0ce34cd07d1283cc72906d1fe78037d49adae455600a5560c060405260808190527f424f524f4e47545300000000000000000000000000000000000000000000000060a09081526101ca91600d9190610284565b50604051610a89380380610a898339810160405280516020820151600b8054600160a060020a03191633179055670de0b6b3a76400008202600c8190559192019061021e9082640100000000610244810204565b151561022957600080fd5b805161023c90600d906020840190610284565b50505061031f565b60408051818152606080820183526000929091906020820161080080388339019050509050828152836020820152600054604082a1602001519392505050565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f106102c557805160ff19168380011785556102f2565b828001600101855582156102f2579182015b828111156102f25782518255916020019190600101906102d7565b506102fe929150610302565b5090565b61031c91905b808211156102fe5760008155600101610308565b90565b61075b8061032e6000396000f30060806040526004361061008d5763ffffffff7c010000000000000000000000000000000000000000000000000000000060003504166318160ddd811461009257806334ce10c4146100b95780636f26d98f146100e45780637b61c320146101075780638da5cb5b146101915780639201de55146101c2578063a9059cbb146101da578063f2fde38b146101fe575b600080fd5b34801561009e57600080fd5b506100a761021f565b60408051918252519081900360200190f35b3480156100c557600080fd5b506100ce610225565b6040805160ff9092168252519081900360200190f35b3480156100f057600080fd5b50610105600160a060020a036004351661022a565b005b34801561011357600080fd5b5061011c61027b565b6040805160208082528351818301528351919283929083019185019080838360005b8381101561015657818101518382015260200161013e565b50505050905090810190601f1680156101835780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b34801561019d57600080fd5b506101a6610309565b60408051600160a060020a039092168252519081900360200190f35b3480156101ce57600080fd5b5061011c600435610318565b3480156101e657600080fd5b50610105600160a060020a03600435166024356104d1565b34801561020a57600080fd5b50610105600160a060020a036004351661062f565b600c5490565b601290565b600b54600160a060020a0316331461024157600080fd5b604051600160a060020a03821690303180156108fc02916000818181858888f19350505050158015610277573d6000803e3d6000fd5b5050565b600d805460408051602060026001851615610100026000190190941693909304601f810184900484028201840190925281815292918301828280156103015780601f106102d657610100808354040283529160200191610301565b820191906000526020600020905b8154815290600101906020018083116102e457829003601f168201915b505050505081565b600b54600160a060020a031681565b60408051602080825281830190925260609160009183918391829184919080820161040080388339019050509350600092505b60208310156103e7576008830260020a870291507fff000000000000000000000000000000000000000000000000000000000000008216156103d15781848681518110151561039657fe5b9060200101907effffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff1916908160001a9053506001909401936103dc565b84156103dc576103e7565b60019092019161034b565b846040519080825280601f01601f191660200182016040528015610415578160200160208202803883390190505b509050600092505b848310156104c757838381518110151561043357fe5b9060200101517f010000000000000000000000000000000000000000000000000000000000000090047f010000000000000000000000000000000000000000000000000000000000000002818481518110151561048c57fe5b9060200101907effffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff1916908160001a90535060019092019161041d565b9695505050505050565b600b54600160a060020a031633146104e857600080fd5b600d805460408051602060026001851615610100026000190190941693909304601f8101849004840282018401909252818152849361058093919290918301828280156105765780601f1061054b57610100808354040283529160200191610576565b820191906000526020600020905b81548152906001019060200180831161055957829003601f168201915b5050505050610675565b101561058b57600080fd5b600d805460408051602060026001851615610100026000190190941693909304601f8101849004840282018401909252818152610624938693919290918301828280156106195780601f106105ee57610100808354040283529160200191610619565b820191906000526020600020905b8154815290600101906020018083116105fc57829003601f168201915b5050505050836106ac565b151561027757600080fd5b600b54600160a060020a0316331461064657600080fd5b600b805473ffffffffffffffffffffffffffffffffffffffff1916600160a060020a0392909216919091179055565b6040805160208082528183019092526000916060919080820161040080388339019050509050828152600154602082a15192915050565b60006106cf848484602060405190810160405280600081525060006001026106d7565b949350505050565b6040805160a080825260c0820190925260009160609190602082016114008038833901905050905086815285602082015284604082015283606082015282608082015260025460a082a16080015196955050505050505600a165627a7a723058204e61bd79f32edb422115d7c6c29c2191ce109cfe1c74fae191fdd21b2873b0050029', 
     gas: '4700000',
     value: web3.toTa(1)
   }, function (e, contract){
    console.log(e, contract);
    if (typeof contract.address !== 'undefined') {
         console.log('Contract mined! address: ' + contract.address + ' transactionHash: ' + contract.transactionHash);
    }
 })
```

把以上修改好的命令拷贝到`attach`的`console`里，执行。可以看到暂时没有结果。只是输出一个提示

```text
null [object Object]
undefined
```

暂时不用担心，交易打包后才会有地址输出出来。第一个终端里，会输出打包的情况。

![&#x4EA4;&#x6613;&#x88AB;&#x6253;&#x5305;&#x63D0;&#x4EA4;](.gitbook/assets/image%20%283%29.png)

交易提交以后，第二个终端的地址就得到了。这个地址是非常非常重要的，一定要保存好。（同样需要保存好的还有源代码、接口文档。）

![&#x5408;&#x7EA6;&#x5730;&#x5740;](.gitbook/assets/image%20%281%29.png)

这个币就发行成功了。接下来就可以使用它了。

## 使用

### 转账

此时查询余额，你会发现所有的币都还在合约地址，而不在`Owner`地址上。这样用起来就很不方便了。

![&#x5E01;&#x90FD;&#x5728;&#x5408;&#x7EA6;&#x4E0A;](.gitbook/assets/image%20%286%29.png)

因此，我们把它们全部转到`Owner`地址上来

```javascript
borongts.transfer(sero.accounts[0],web3.toTa(1024),{from:sero.accounts[0]})
```

交易完成后，重新查询余额，发现全部余额都已经到`Owner`地址了。

![&#x5168;&#x90E8;&#x8F6C;&#x5165;Owner](.gitbook/assets/image%20%282%29.png)

我们可以试着转给其他人了。

```javascript
sero.sendTransaction({from:sero.accounts[0],to:sero.accounts[1],cy:"BORONGTS",value:web3.toTa(10)})
```

![&#x8F6C;&#x7ED9;&#x4ED6;&#x4EBA;](.gitbook/assets/image%20%288%29.png)

### 再次使用合约

你可能注意到了，以上示例在发布完成后，获得了一个合约的实例`borongts`，然后通过调用`borongts`的`transfer`函数，把全部币转入了`Owner`。那么关闭了控制台以后，下次该如何获得这个实例，以便再次调用合约中的函数呢？

答案是使用`at`函数。

```javascript
var borongtsContract = web3.sero.contract([{"constant":true,"inputs":[],"name":"totalSupply","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"getDecimal","outputs":[{"name":"","type":"uint8"}],"payable":false,"stateMutability":"pure","type":"function"},{"constant":false,"inputs":[{"name":"_to","type":"address"}],"name":"reclaimSero","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"tokenSymbol","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"owner","outputs":[{"name":"","type":"address"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[{"name":"x","type":"bytes32"}],"name":"bytes32ToString","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"pure","type":"function"},{"constant":false,"inputs":[{"name":"_to","type":"address"},{"name":"_value","type":"uint256"}],"name":"transfer","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"newOwner","type":"address"}],"name":"transferOwnership","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"inputs":[{"name":"initSupply","type":"uint256"},{"name":"tokenName","type":"string"}],"payable":true,"stateMutability":"payable","type":"constructor"}]);
var brts = borongtsContract.at('tqt82x8DcHyj12x4mywqBczPjcFjBnCN7WrXfMYSdPto8FB1ybmmqsv4GM6QSVoQb2RQQQcjqocdiegorTaCBoy');
```

这样通过`ABI`和地址，就可以方便的获得对象实例，调用其中的函数了。

![&#x518D;&#x6B21;&#x8C03;&#x7528;&#x5408;&#x7EA6;](.gitbook/assets/image.png)

## 费用

SERO被设计成发行收费的经济模型。目前收费的情况如下：

* Dev网络
  * 统一收费1 SERO
* Beta网络
  * 1个字母：100万 SERO
  * 2个字母：30万 SERO
  * 3个字母：10万 SERO
  * 4个字母：1000 SERO
  * 其他：1 SERO

