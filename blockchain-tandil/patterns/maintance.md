# Maintance
Maintenance is a group of patterns that provide mechanismsfor live operating contracts. In contrast to ordinary distributedapplications,  which  can  be  updated  when  bugs  are  detected,smart  contracts  are  irreversible  and  immutable.  This  meansthat  there  is  no  way  to  update  a  smart  contract,  other  thanwriting  an  improved  version  that  is  then  deployed  as  newcontract.

## Data segregation

**Problem:** Contract data and its logic are usually kept in the same contract,leading  to  a  closely  entangled  coupling.  Once  a  contract  is  replaced  bya  newer  version,  the  former  contract  data  must  be  migrated  to  the  newcontract version.

**Solution:** Decouple  the  data  from  the  operational  logic  into  separatecontracts.

The  data  segregation  pattern  separates  contract  logic  fromits  underlying  data.  Segregation  promotes  the  separation  ofconcerns  and  mimics  a  layered  design  (e.g.  logic  layer,  datalayer).  Following  this  principle  avoids  costly  data  migrationswhen  code  functionality  changes.  Meaning  a  new  contractversion  would  not  have  to  recreate  all  of  the  existing  datacontained in the previous contract.

It is favourable to design the storage contract very generic sothat once it is created, it can store and access different typesof data with the help of setter and getter methods.

```solidity
pragma solidity ˆ0.4.17;
contract DataStorage {
  mapping(bytes32 => uint) uintStorage;
  function getUintValue(bytes32 key) public constantreturns (uint) {
    return uintStorage[key];
  }
  
  function setUintValue(bytes32 key, uint value) public {
    uintStorage[key] = value;
  }
}
```

```solidity
pragma solidity ˆ0.4.17;
import "./DataStorage.sol";
contract Logic {
  DataStorage dataStorage;
  function Logic(address _address) public {
    dataStorage = DataStorage(_address);
  }
  
  function f() public {
    bytes32 key = keccak256("emergency");
    dataStorage.setUintValue(key, 911);
    dataStorage.getUintValue(key);
  }
}
```

## Satellite

**Problem:** Contracts  are  immutable.  Changing  contract  functionality  re-quires the deployment of a new contract.

**Solution:** Outsource functional units that are likely to change into separateso-called satellite contracts and use a reference to these contracts in orderto utilize needed functionality.

The satellite pattern allows to modify and replace contractfunctionality. This is achieved through the creation of separatesatellite contracts that encapsulate certain contract functional-ity.  The  addresses  of  these  satellite  contracts  are  stored  in  abase contract. This contract can then can call out to the satellitecontracts when it needs to reference certain functionalities, byusing  the  stored  address  pointers.  If  this  pattern  is  properlyimplemented, modifying functionality is as simple as creatingnew satellite contracts and changing the corresponding satelliteaddresses.

```solidity
pragma solidity ˆ0.4.17;
contract Satellite {
  function calculateVariable() public pure returns (uint){
    // calculate var
    return 2*3;
  }
}
```

```solidity
pragma solidity ˆ0.4.17;
import "../../authorization/Ownership.sol";
import "./Satellite.sol";
contract Base is Owned {
  uint public variable;
  address satelliteAddress;
  function setVariable() public onlyOwner {
    Satellite s = Satellite(satelliteAddress);
    variable = s.calculateVariable();
  }
  
  function updateSatelliteAddress(address _address) publiconlyOwner {
    satelliteAddress = _address;
  }
}
```

## Contract Register

Before invoking it, the address of the latest version of a smart contract is located by looking up its
name on a contract registry.

As any software application, blockchain-based applications need to be upgraded to new versions. To
do so, the on-chain functions de�ned in smart contracts need to be updated to �x bugs as well as to ful�l new
requirements.

**Problem:** Contract  participants  must  be  referred  to  the  latest  contract version.

**Solution:** Let  contract  participants  pro-actively  query  the  latest  contractaddress  through  a  register  contract  that  returns  the  address  of  the  most recent version.

The  register  pattern  is  an  approach  to  handle  the  updateprocess  of  a  contract.  The  pattern  keeps  track  of  differentversions  (addresses)  of  a  contract  and  points  on  request  tothe  latest  one. Before interacting with a contract, a user would always have to querythe  register  for  the  contract’s  latest  address.  When  followingthis update approach, it is also important to determine how tohandle existing contract data, when an old contract version isreplaced. An alternative solution to point to the latest contractaddress would be to utilize the Ethereum Name Service (ENS).It  is  a  register  that  enables  a  secure  and  decentralised  wayto  resolve  human-readable  names,  like  ’mycontract.eth’,  intomachine-readable identifiers, including Ethereum addresses.

```solidity
pragma solidity ˆ0.4.17;
import "../authorization/Ownership.sol";
contract Register is Owned {
  address backendContract;
  address[] previousBackends;
  
  function Register() public {
    owner = msg.sender;
  }
  
  function changeBackend(address newBackend) publiconlyOwner() returns (bool) {
    if(newBackend != backendContract) {
      previousBackends.push(backendContract);
      backendContract = newBackend;
      return true;
    }
  return false;
  }
}
```

## Contract Relay

**Problem:** Contract  participants  must  be  referred  to  the  latest  contractversion.

**Solution:** Contract   participants   always   interact   with   the   same   proxycontract that relays all requests to the most recent contract version.

A  relay  is  another  approach  to  handle  the  update  processof  a  contract.  The  relay  pattern  provides  a  method  to  updatea  contract  to  a  newer  version  while  keeping  the  old  contractaddress. This is achieved by using a kind of proxy contract thatforwards  calls  and  data  to  the  latest  version  of  the  contract. 
his approach can forward function callsincluding  their  arguments,  but  cannot  return  result  values.Another  drawback  of  this  approach  is  that  the  data  storage layout  needs  to  be  consistent  in  newer  contract  versions,otherwise data may be corrupted.

```solidity
pragma solidity ˆ0.4.17;
import "../authorization/Ownership.sol";
contract Relay is Owned {
  address public currentVersion;
  function Relay(address initAddr) public {
    currentVersion = initAddr;
    owner = msg.sender;
  }
  
  function changeContract(address newVersion) publiconlyOwner() {
    currentVersion = newVersion;
  }
  
  // fallback 
  functionfunction() public {
    require(currentVersion.delegatecall(msg.data));
  }
}
```

## Factory Contract

An on-chain template contract is used as a factory that generates contract instances from the template.

Applications based on blockchain might need to use multiple instances of a standard contract with
customization. Each contract instance is created by instantiating a contract template. For example, in a business
process management system, each of the business process instances might be represented by a smart contract
being generated from a contract template representing the business process model. The template can be
stored off-chain in a code repository, or on-chain, within its own smart contract.

**Problem:** Keeping the contract template o�-chain cannot guarantee consistency between different smart contract
instances created from the same template because the source code of the template can be independently modified.

**Solution:** Smart contracts are created from a contract factory deployed on blockchain. The factory contract is
deployed once from the off-chain source code. The factory may contain the de�nition of multiple smart contracts.
Smart contract instances are generated by passing parameters to the contract factory to instantiate customized
smart contract instances. Factory contract is analogous to a Class in an object-oriented programming language.
Every transaction that generates a smart contract instance essentially instantiates an object of the factory contract
class. This contract instance (the object) will maintain its own properties independently of the other instances
but with a structure consistent with its original template. 

```solidity
// TODO
```


