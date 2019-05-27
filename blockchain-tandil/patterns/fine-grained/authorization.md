# Authorizrion

Authorization  is  a  group  of  patterns  that  control  access  tosmart contract functions and provide basic authorization con-trol, which simplify the implementation of “user permissions”.

## Ownership

**Problem:** By  default  any  party  can  call  a  contract  method,  but  it  must be  ensured  that  sensitive  contract  methods  can  only  be  executed  by  the owner of a contract.

**Solution:** Store the contract creator’s address as owner of a contract andrestrict method execution dependent on the callers address.

```solidity
pragma solidity ˆ0.4.17;

contract Owned {
  address public owner;
  event LogOwnershipTransferred(address indexedpreviousOwner, address indexed newOwner);
  modifier onlyOwner() {
    require(msg.sender == owner);
    _;    
  }
  
  function Owned() public {
    owner = msg.sender;
  }

  function transferOwnership(address newOwner) public onlyOwner {
    require(newOwner != address(0));
    LogOwnershipTransferred(owner, newOwner);
    owner = newOwner;
  }
}
```
## Access restriction

**Problem:** By  default  a  contract  method  is  executed  without  any  preconditions being checked, but it is desired that the execution is only allowed if certain requirements are met.

**Solution:** Define generally  applicable  modifiers  that  check  the  desired requirements and apply these modifiers in the function definition.

Since  there  is  no  built  in  mechanism  to  control  executionprivileges, a common pattern is to restrict function execution.It  is  often  required  that  functions  should  only  be  executedbased on the presence of certain prerequisites. These can referto different categories, such as temporal conditions, caller andtransaction info, or other requirements that need to be checkedprior  a  function  execution. 

```solidity
pragma solidity ˆ0.4.17;
import "./Ownership.sol";
contract AccessRestriction is Owned {
  uint public creationTime = now;
  modifier onlyBefore(uint _time) {
    require(now < _time); 
    _;
  }
  
  modifier onlyAfter(uint _time) {
    require(now > _time);
    _;
    }
  modifier onlyBy(address account) {
    require(msg.sender == account); 
    _;
  }
  
  modifier condition(bool _condition) {
    require(_condition); 
    _;
  }
  
  modifier minAmount(uint _amount) {
    require(msg.value >= _amount); 
    _;
  }
  
  function f() payable onlyAfter(creationTime + 1 minutes) onlyBy(owner) minAmount(2 ether) condition(msg.sender.balance >= 50 ether) {
    // some code
  }
}
```