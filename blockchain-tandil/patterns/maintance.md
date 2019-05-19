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

**Problem:** Contract  participants  must  be  referred  to  the  latest  contractversion.

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