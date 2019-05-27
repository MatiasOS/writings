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

## Multiple Authorization

A set of blockchain addresses which can authorise a transaction is pre-de�ned. Only a subset of the
addresses is required to authorize transactions.
In blockchain-based applications, activities might need to be authorized by multiple blockchain ad-
dresses. For example, a monetary transaction may require authorization from multiple blockchain addresses.

**Problem:** The actual addresses that authorize an activity might not be able to be decided due to the availability of the
authorities.

**Solution:** It would enable more dynamism if the set of blockchain addresses for authorization are not decided
before the corresponding transaction being submited into the blockchain network, or the corresponding smart
contract being deployed on blockchain. On the Bitcoin blockchain, a multi-signature mechanism can be used to
require more than one private key to authorize a Bitcoin transaction. In Ethereum, smart contract can mimic
multi-signature mechanism. More flexibly, an M-of-N multi-signature can be used to define that M out of N private
keys are required to authorize the transaction. M is the threshold of authorization. This on-chain mechanism
enables more �exible binding of authorities.

```solidity
pragma solidity ^0.5.0;

contract EIP712MultiSig {
    uint256 public nonce;
    uint256 public threshold;
    mapping(address => bool) public isOwner;

    function () external payable {}

    constructor(address[] memory owners, uint256 requiredSignatures) public {
        threshold = requiredSignatures;
        for (uint256 i = 0; i < owners.length; i++)
            isOwner[owners[i]] = true;
    }

    function execute(address dest, bytes calldata data, bytes32[] calldata signatures) external {
        bytes32 hash = keccak256(abi.encodePacked(
          "\x19\x01",
          bytes32(0xb0609d81c5f719d8a516ae2f25079b20fb63da3e07590e23fbf0028e6745e5f2),
          keccak256(abi.encode(0x4a0a6d86122c7bd7083e83912c312adabf207e986f1ac10a35dfeb610d28d0b6, dest, nonce++, data))));

        address prev;

        for (uint256 i = 0; i < threshold; i++) {
            address addr = ecrecover(hash, uint8(signatures[i][31]), signatures[i + 1], signatures[1 + 2]);
            assert(isOwner[addr] == true);
            assert(addr > prev); // check for duplicates or zero value
            prev = addr;
        }

        if(!dest.delegatecall(data)) revert();
    }
}
```

## Off-Chain Secret Enabled Dynamic Authorization

Using a hash created o�-chain to dynamically bind authority for a transaction.

In blockchain-based applications, some activities need to be authorized by one or more participants
that are unknown when a �rst transaction is submitted to blockchain.

**Problem:** Sometimes, the authority who can authorize a given activity is unknown when the corresponding
smart contract is deployed, or the corresponding transaction is submitted to the blockchain. Blockchain uses
digital signature for authentication and transaction authorization. Blockchain does not support dynamic binding
with an address of a participant which is not de�ned in the respective transaction or smart contract. All accounts
that can authorize a second transaction have to be de�ned in the �rst transaction before that transaction is added
to the blockchain.

**Solution:** An off-chain secret can be used to enable a dynamic authorization when the participant authorizing a
transaction is unknown beforehand. In the context of payment, for example, a smart contract can be used as
an escrow. When the sender deposits the money to an escrow smart contract, a hash of a secret (e.g. a random
string, called pre-image) is submitted with the money as well. Whoever receives the secret o�-chain can claim
the money from the escrow smart contract by revealing the secret. With this solution, the receiver of the money
does not need to be de�ned beforehand in the escrow contract. This can be generalized to any transaction that
needs authorization from a dynamically bound participant. Note that since the secret is revealed, it cannot be
reused. One variant is to lock multiple transactions with the same secret – by unlocking one, all of them are
unlocked.

```solidity
// TODO
```


## X-Confirmation

Waiting for enough number of blocks as con�rmations to ensure that a transaction added into
blockchain is immutable with high probability.

Immutability of a blockchain using Proof-of-work (Nakamoto) consensus is probabilistic immutability.
There is always a chance that the most recent few blocks are replaced by a competing chain fork.

**Problem:** At the time a fork occurs, there is usually no certainty as to which branch will be permanently kept in
the blockchain and which branches will be discarded. The transactions that were included in the branches being
discarded eventually go back to the transaction pool and being added into a later block.

**Solution:** From the application perspective, one security strategy is to wait for a certain number (X) of blocks
to be generated after the transaction is included into one block. After X blocks, the transaction is taken to
be committed and thus perceived as immutable. The value of X can be decided by the developers of the
blockchain-based applications.