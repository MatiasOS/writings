# Action and control 

Action  and  Control  is  a  group  of  patterns  that  providemechanisms for typical operational tasks.

## Pull payment

**Problem:** When a contract sends funds to another party, the send operationcan fail.\
**Solution:** Let the receiver of a payment withdraw the funds.

Currently,  there  are  three  different  methods  to  transferfunds  in  Solidity.  These  areaddress.send(),address.transfer(),  andaddress.call.value()().  If  the  pay-ment  recipient  is  a  contract,  calling  theses  methods  triggersthe  execution  of  a  so-called  fallback  function  in  the  receiver contract.  Per  definition,  the  fallback  function  is  a  name-and  parameterless  function,  that  is  called  when  the  functionsignature  does  not  match  any  of  the  available  functions  ina  Solidity  contract.  Sincesend()specifies  a  blank  functionsignature, it will always trigger the fallback function if it exists.x.transfer(y)is equivalent torequire(x.send(y))anddefines a maximum stipend of 2,300 gas, given to the receivercontract for execution, which is currently only enough to logan event.address.call.value()()gives all available gasto the receiving contract for execution, which makes this typeof value transfer unsafe against re-entrancy. So, the differencebetweensend()andaddress.call.value()()is  howmuch  gas  is  made  available  to  the  fallback  function  in  thereceiving contract, thereby controlling the risk.

```solidity
pragma solidity ˆ0.4.17;
contract Auction {
  address public highestBidder;
  uint highestBid;
  function bid() public payable {
    require(msg.value >= highestBid);
    if (highestBidder != 0) {
      // if call fails causing a rollback,
      // no one else can 
      bidhighestBidder.transfer(highestBid);
    }
    highestBidder = msg.sender;
    highestBid = msg.value;
  }
}
```

```solidity
pragma solidity ˆ0.4.17;
contract Auction {
  address public highestBidder;
  uint highestBid;
  mapping(address => uint) refunds;
  function bid() public payable {
    require(msg.value >= highestBid);
    if (highestBidder != 0) {
      // record the underlying bid to be refund
      refunds[highestBidder] += highestBid;
    }
    highestBidder = msg.sender;
    highestBid = msg.value;
  }
  
  function withdrawRefund() public {
    uint refund = refunds[msg.sender];
    refunds[msg.sender] = 0;
    msg.sender.transfer(refund);
  }
}
```

## State machine

**Problem:** An application scenario implicates different behavioural stagesand transitions.\
**Solution:** Apply  a  state  machine  to  model  and  represent  different  be-havioural contract stages and their transitions.

```solidity
pragma solidity ˆ0.4.17;
contract DepositLock {
  enum Stages {
    AcceptingDeposits,
    FreezingDeposits,
    ReleasingDeposits
  }

  Stages public stage = Stages.AcceptingDeposits;
  uint public creationTime = now;
  mapping (address => uint) balances;
  modifier atStage(Stages _stage) {
    require(stage == _stage);
    _;
  }

  modifier timedTransitions() {
    if (stage == Stages.AcceptingDeposits && now >=creationTime + 1 days)
      nextStage();
    if (stage == Stages.FreezingDeposits && now >=creationTime + 8 days)
      nextStage();
    _;
  }

  function nextStage() internal {
    stage = Stages(uint(stage) + 1);
  }

  function deposit() public payable timedTransitionsatStage(Stages.AcceptingDeposits) {
    balances[msg.sender] += msg.value;
  }

  function withdraw() public timedTransitions atStage(Stages.ReleasingDeposits) {
    uint amount = balances[msg.sender];
    balances[msg.sender] = 0;
    msg.sender.transfer(amount);
  }
}
```
A contract  based  on  a  state  machine  to  represent  a  deposit  lock,which accepts deposits for a period of one day and releases them after sevendays.

## Commit and reveal

**Problem:** All   data   and   every   transaction   is   publicly   visible   on   theblockchain, but an application scenario requires that contract interactions,specifically submitted parameter values, are treated confidentially.

**Solution:** Apply a commitment scheme to ensure that a value submissionis binding and concealed until a consolidation phase runs out, after whichthe value is revealed, and it is publicly verifiable that the value remainedunchanged.

```solidity
pragma solidity ˆ0.4.17;
contract CommitReveal {

  struct Commit {
    string choice; 
    string secret; 
    stringstatus;
  }
  mapping(address => mapping(bytes32 => Commit)) 
  publicuserCommits;
  event LogCommit(bytes32, address);
  event LogReveal(bytes32, address, string, string);
  function CommitReveal() public {} // Constructor?

  function commit(bytes32 _commit) public returns (boolsuccess) {
    var userCommit = userCommits[msg.sender][_commit];
    if(bytes(userCommit.status).length != 0) {
      return false; // commit has been used before
    }
    userCommit.status = "c"; // comitted
    LogCommit(_commit, msg.sender);
    return true;
  }
  
  function reveal(string _choice, string _secret, bytes32_commit) public returns (bool success) {
    var userCommit = userCommits[msg.sender][_commit];
    bytes memory bytesStatus = bytes(userCommit.status);
    if(bytesStatus.length == 0) {
      return false; // choice not committed before
    } else if (bytesStatus[0] == "r") {
      return false; // choice already revealed
    }
    if (_commit != keccak256(_choice, _secret)) {
      return false; // hash does not match commit
    }
    userCommit.choice = _choice;
    
    userCommit.secret = _secret;userCommit.status = "r"; // revealed
    LogReveal(_commit, msg.sender, _choice, _secret);
    return true;
  }
  
  function traceCommit(address _address, bytes32 _commit)public view returns (string choice, string secret,string status) {
    var userCommit = userCommits[_address][_commit];
    require(bytes(userCommit.status)[0] == "r");
    return (userCommit.choice, userCommit.secret,userCommit.status);
  }
}
```
A contract that allows a party to commit to a choice and reveal itat a later point in time, traceable for anyone.

## Oracle (Data provider)

**Problem:** An  application  scenario  requires  knowledge  contained  outsidethe blockchain, but Ethereum contracts cannot directly acquire informationfrom  the  outside  world.  On  the  contrary,  they  rely  on  the  outside  worldpushing information into the network.\
**Solution:** Request external data through an oracle service that is connectedto the outside world and acts as a data carrier.\

Ethereum contracts run within their own ecosystem, wherethey communicate with each other, but external data can onlyenter the system through outside interaction via a transaction(by  passing  data  to  a  method).  This  is  a  drawback,  becausemany contract use cases depend on external knowledge outsidethe  blockchain  (e.g.  price  feeds).
A  shortcoming  of  this  solution  is  that  oracles  contradictthe  blockchain  theorem  of  a  decentralized  network,  becausecontracts utilizing an oracle rely on a single party or group tobe honest. Currently operating oracle services addressthis  shortcoming  by  accompanying  the  resulting  data  with  aproof of authenticity. It should be noted, that an oracle has topay for the callback invocation, thus an oracle usually requirespayment of an oracle fee, plus Ether necessary for the callback.

```solidity
pragma solidity ˆ0.4.17;
contract Oracle {
  address knownSource = 0x123...; // known source
  struct Request {
    bytes data;
    function(bytes memory) external callback;
  }
  Request[] requests;
  event NewRequest(uint);
  
  modifier onlyBy(address account) {
    require(msg.sender == account);
    _;
  }
  
  function query(bytes data, function(bytes memory)external callback) public {
    requests.push(Request(data, callback));
    NewRequest(requests.length - 1);
  }
  
  // invoked by outside world
  function reply(uint requestID, bytes response) publiconlyBy(knownSource) {
    requests[requestID].callback(response);
  }
}
```