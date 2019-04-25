# 部署合约

## 源码

{% code-tabs %}
{% code-tabs-item title="SDET.sol" %}
```text
pragma solidity ^0.4.16;

import "browser/seroInterface.sol";

// address
// 

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

contract SDET is SeroInterface, HasOwner {
    
    using SafeMath for uint256;
    // Public variables of the token
    uint8 constant DECIMALS = 18;
    uint256 public totalSupply;
    string public tokenSymbol = "SDET";
    
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

