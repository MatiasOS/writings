# Life cicle

Lifecycle is a group of patterns that control the creation anddestruction of smart contracts.

## Mortal
**Problem:** A deployed contract will exist as long as the Ethereum networkexists.  If  a  contract’s  lifetime  is  over,  it  must  be  possible  to  destroy  a contract and stop it from operating.

**Solution:** Use a selfdestruct call within a method that does a preliminaryauthorization check of the invoking party.

A  contract  is  defined  by  its  creator,  but  the  execution,and  subsequently  the  services  it  offers  are  provided  by  theEthereum  network  itself.  Thus,  a  contract  will  exist  and  beexecutable as long as the whole network exists, and will onlydisappear  if  it  was  programmed  to  self  destruct.  Mortal  is  apattern  that  enables  the  creator  of  a  contract  to  destroy  it.The  pattern  uses  a  modifier  to  ensure  that  only  the  ownerof  the  contract  can  execute  theselfdestructoperation,which sends the remaining Ether stored within the contract toa  designated  target  address  (provided  as  argument)  and  thenthe storage and code is cleared from the current state.

```solidity
pragma solidity ˆ0.4.17;
import "../authorization/Ownership.sol";
contract Mortal is Owned {

  function destroy() public onlyOwner {
    selfdestruct(owner);
  }
  
  function destroyAndSend(address recipient) publiconlyOwner {
    selfdestruct(recipient);
  }
}
```

## Automatic deprecation

**Problem:** A usage scenario requires a temporal constraint defining a pointin time when functions become deprecated.

**Solution:** Define  an  expiration  time  and  apply  modifiers  in  functiondefinitions  to  disable  function  execution  if  the  expiration  date  has been reached.

Automatic deprecation is a pattern that allows to automati-cally prohibit the execution of functions after a specific time
period has elapsed.

```solidity
pragma solidity ˆ0.4.17;
contract AutoDeprecate {
  uint expires;
  function AutoDeprecate(uint _days) public {
    expires = now + _days*1 days;
  }
  
  function expired() internal view returns (bool) {
    return now > expires;
  }
  
  modifier willDeprecate() {
    require(!expired());
    _;
  }
  
  modifier whenDeprecated() {
    require(expired());
    _;
  }
  
  function deposit() public payable willDeprecate {
    // some code
  }
  
  function withdraw() public view whenDeprecated {
    // some code
  }
}
```