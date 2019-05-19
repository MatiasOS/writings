# Security

Security is a group of patterns that introduce safety measuresto mitigate damage and assure a reliable contract execution.

## Check effect intereaction

**Problem:** When  a  contract  calls  another  contract,  it  hands  over  controlto that other contract. The called contract can then, in turn, re-enter thecontract by which it was called and try to manipulate its state or hijackthe control flow through malicious code.

**Solution:** Follow a recommended functional code order, in which calls toexternal contracts are always the last step, to reduce the attack surface ofa contract being manipulated by its own externally called contracts.

The Checks-Effects-Interaction pattern is fundamental forcoding  functions  and  describes  how  function  code  shouldbe  structured  to  avoid  side  effects  and  unwanted  executionbehaviour.  It  defines  a  certain  order  of  actions:  First,  checkall  the  preconditions,  then  make  changes  to  the  contract’sstate, and finally interact with other contracts. Hence its nameis  “Checks-Effects-Interactions  Pattern”.  According  to  thisprinciple, interactions with other contracts should be, wheneverpossible, the very last step in any function.
The reason being, that as soon as a contract interacts with another contract, including a transfer of Ether, it hands over thecontrol to that other contract. This allows the called contract toexecute potentially harmful actions.  For example, a so-calledre-entrancy  attack,  where  the  called  contract  calls  back  thecurrent  contract,  before  the  first  invocation  of  the  functioncontaining the call, was finished. This can lead to an unwantedexecution behaviour of functions, modifying the state variablesto  unexpected  values  or  causing  operations  (e.g.  sending  offunds)  to  be  performed  multiple  times.
he re-entrancy attack is especially harmfulwhen  using  low  leveladdress.call,  which  forwards  allremaining gas by default, giving the called contract more roomfor  potentially  malicious  actions.  Therefore,  the  use  of  lowleveladdress.callshould be avoided whenever possible. Forsending fundsaddress.send()andaddress.transfer()should be preferred, these functions minimize the risk of re-entrancy through limited gas forwarding. While these methodsstill trigger code execution, the called contract is only given astipend of 2,300 gas, which is currently only enough to log anevent.

```solidity
function auctionEnd() public {
  // 1. Checks
  require(now >= auctionEnd);
  require(!ended);
  // 2. Effect
  sended = true;
  // 3. Interaction
  beneficiary.transfer(highestBid);
}
```

```solidity
mapping (address => uint) balances;
function withdrawBalance() public {
  uint amount = balances[msg.sender];
  require(msg.sender.call.value(amount)()); // caller’scode is executed and can re-enter withdrawBalanceagain
  balances[msg.sender] = 0; // INSECURE - user’s balancemust be reset before the external call
}
```
## Emergency stop

**Problem:** Since  a  deployed  contract  is  executed  autonomously  on  theEthereum network, there is no option to halt its execution in case of amajor bug or security issue.

**Solution:** Incorporate an emergency stop functionality into the contract thatcan be triggered by an authenticated party to disable sensitive functions.

Reliably  working  contracts  may  contain  bugs  that  areyet  unknown,  until  revealed  by  an  adversary  attack.  Onecountermeasure  and  a  quick  response  to  such  attacks  areemergency stops or circuit breakers. They stop the executionof a contract or its parts when certain conditions are met. Arecommended scenario would be, that once a bug is detected, allcritical functions would be halted, leaving only the possibilityto  withdraw  funds. The ability to fire an emergency stop could be either given to a certain party, or handled throughthe implementation of a rule set.

```solidity
pragma solidity ^0.4.17;
import "../authorization/Ownership.sol";
contract EmergencyStop is Owned {
  bool public contractStopped = false;
  modifier haltInEmergency {
    if (!contractStopped) _;
  }
  
  modifier enableInEmergency {
    if (contractStopped) _;
  }
    
  function toggleContractStopped() public onlyOwner {
    contractStopped = !contractStopped;
  }
  
  function deposit() public payable haltInEmergency {
    // some code
  }
  
  function withdraw() public view enableInEmergency {
    // some code
  }
}
```

## Speed bump

**Problem:** The simultaneous execution of sensitive tasks by a huge numberof parties can bring about the downfall of a contract.

**Solution:** Prolong the completion of sensitive tasks to take steps againstfraudulent activities.

Contract  sensitive  tasks  are  slowed  down  on  purpose,  sowhen  malicious  actions  occur,  the  damage  is  restricted  andmore  time  to  counteract  is  available.  An  analogous  realworld example would be a bank run, where a large numberof  customers  withdraw  their  deposits  simultaneously  due  toconcerns about the bank’s solvency. Banks typically counteractby delaying, stopping, or limiting the amount of withdrawals.

```solidity
pragma solidity ^0.4.17;
contract SpeedBump {
  struct Withdrawal {
    uint amount;
    uint requestedAt;
  }
  mapping (address => uint) private balances;
  mapping (address => Withdrawal) private withdrawals;
  uint constant WAIT_PERIOD = 7 days;
  function deposit() public payable {
    if(!(withdrawals[msg.sender].amount > 0)) balances[msg.sender] += msg.value;
  }
  
  function requestWithdrawal() public {
    if (balances[msg.sender] > 0) {
      uint amountToWithdraw = balances[msg.sender];
      balances[msg.sender] = 0;
      withdrawals[msg.sender] = Withdrawal({
        amount: amountToWithdraw,
        requestedAt: now
      });
    }
  }
  
  function withdraw() public {
    if(withdrawals[msg.sender].amount > 0 && now >withdrawals[msg.sender].requestedAt + WAIT_PERIOD){
      uint amount = withdrawals[msg.sender].amount;withdrawals[msg.sender].amount = 0;msg.sender.transfer(amount);
    }
  }
}
```

## Rate limit

**Problem:** A request rush on a certain task is not desired and can hinder the correct operational performance of a contract.

**Solution:** Regulate how often a task can be executed within a period oftime.

A  rate  limit  regulates  how  often  a  function  can  be  calledconsecutively within a specified time interval. This approachmay be used for different reasons. A usage scenario for smartcontracts may be founded on operative considerations, in orderto  control  the  impact  of  (collective)  user  behaviour.  As  anexample  one  might  limit  the  withdrawal  execution  rate  ofa  contract  to  prevent  a  rapid  drainage  of  funds.

```solidity
pragma solidity ^0.4.17;
contract RateLimit {
  uint enabledAt = now;
  modifier enabledEvery(uint t) {
    if (now >= enabledAt) {
      enabledAt = now + t;_;
    }
  }
  
  function f() public enabledEvery(1 minutes) {
    // some code
  }
}
```
## Mutex

**Problem:** Re-entrancy attacks can manipulate the state of a contract and hijack the control flow.

**Solution:** Utilize a mutex to hinder an external call from re-entering itscaller function again.

A mutex (from mutual exclusion) is known as a synchro-nization mechanism in computer science to restrict concurrentaccess to a resource. After re-entrancy attack scenarios emerged,this pattern found its application in smart contracts to protectagainst  recursive  function  calls  from  external  contracts.

```solidity
pragma solidity ^0.4.17;
contract Mutex {
  bool locked;
  modifier noReentrancy() {
    require(!locked);
    locked = true;
    _;
    locked = false;
  }
  // f is protected by a mutex, thus reentrant calls
  // from within msg.sender.call cannot call f again
  function f() noReentrancy public returns (uint) {
    require(msg.sender.call());
    return 1;
  }
}
```
## Balance limit

**Problem:** There  is  always  a  risk  that  a  contract  gets  compromised  dueto bugs in the code or yet unknown security issues within the contractplatform.

**Solution:** Limit the maximum amount of funds at risk held within a contract.

It is generally a good idea to manage the amount of moneyat risk when coding smart contracts. This can be achieved bylimiting the total balance held within a contract. The patternmonitors the contract balance and rejects payments sent along afunction invocation after exceeding a predefined quota, as seenin Listing 8. It should be noted that this approach cannot preventthe admission of forcibly sent Ether, e.g. as beneficiary of aselfdestruct(address)call, or as recipient of a miningreward.

```solidity
pragma solidity ^0.4.17;
contract LimitBalance {
  uint256 public limit;
  function LimitBalance(uint256 value) public {
    limit = value;
  }
  
  modifier limitedPayable() {
    require(this.balance <= limit);
    _;
  }
  
  function deposit() public payable limitedPayable {
    // some code
  }
}
```