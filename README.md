# Egg Supply chain tracking

This project implements supply chain tracking for eggs.
Everything is implemented completely decentralized via Ethereum Smart Contracts and IPFS.

The goal is to allow customers more trust in their purchase. 
In recent years, consumers have become more and more aware of the horrendous conditions in some chicken farms. At the same time, there have been repeated scandals or eggs being wrongfully labeled as organic.


UML diagrams are coded using mermaid.

## 1 . Project Plan

### UML

### Activity diagram
Here's what the activity diagram looks like

```mermaid
    sequenceDiagram
    participant Eggs
    participant Farmer
    participant Distributor
    participant Retailer
    participant Consumer
    Farmer ->> Eggs: harvestItem()
    Farmer ->> Eggs: processItem
    Farmer ->> Eggs: packItem()
    Farmer ->> Eggs: addItem()
    Distributor ->> Farmer: buyItem()
    Farmer ->> Retailer: shipItem()
    Retailer ->> Farmer: recieveItem()
    Consumer ->> Retailer: purchaseItem()
    Eggs ->> Consumer: fetchItem()
    Eggs ->> Consumer: fetchItem()
```

### Class diagrams
Solidity roles contracts
```mermaid
classDiagram
class Roles{
    +String beakColor
    +swim()
    +quack()
}
class ConsumerRole{
    + isConsumer()
    + addConsumer()
    + renounceConsumer()
    # _addConsumer()
    # _removeConsumer()
}
class DistributorRole{
    + isDistributor()
    + addDistributor()
    + renounceDistributor()
    # _addDistributor()
    # _removeDistributor()
}
class FarmerRole{
    + isFarmer()
    + addFarmer()
    + renounceFarmer()
    # _addFarmer()
    # _removeFarmer()
}
class RetailerRole{
    + isRetailer()
    + addRetailer()
    + renounceRetailer()
    # _addRetailer()
    # _removeRetailer()
}

Roles <|-- FarmerRole
Roles <|-- DistributorRole
Roles <|-- RetailerRole
Roles <|-- ConsumerRole
```
Solidity SupplyChain contract
```mermaid
classDiagram
class Item {
    uint    sku
    uint    upc
    address payable ownerID
    address payable originFarmerID
    string  originFarmName
    string  originFarmInformation
    string  originFarmLatitude
    string  originFarmLongitude
    uint    productID
    string  productNotes
    uint    productPrice
    State   itemState
    address distributorID
    address retailerID
    address payable consumerID
}
class SupplyChain {
  address owner
  uint upc
  uint sku
  mapping items
  mapping itemsHistory
  enum State
  State defaultState
  + kill()
  - _make_payable()
  + harvestItem()
  + processItem()
  + packItem()
  + sellItem()
  + buyItem()
  + shipItem()
  + recieveItem()
  + purchaseItem()
  + fetchItemBufferOne()
  + fetchItemBufferTwo()
}
SupplyChain -- Item: declares
```

### Sequence diagram

```mermaid
sequenceDiagram
    participant Farmer
    participant Distributer
    participant Retailer
    participant Consumer
    participant FrontEnd
    participant SupplyChainContract
    participant ContractData
    
    Farmer ->> FrontEnd: Harverst Item
    activate Farmer
    FrontEnd ->> SupplyChainContract: Register new Item
    activate FrontEnd
    SupplyChainContract ->> FrontEnd: Display transaction ID
    activate SupplyChainContract

    %% Process
    Farmer ->> FrontEnd: Process Item
    FrontEnd ->> SupplyChainContract: Process Item
    SupplyChainContract ->> ContractData: Verify
    activate ContractData
    alt successful
        ContractData-->>SupplyChainContract: OK
        SupplyChainContract ->> ContractData: UpdateProduct State
        SupplyChainContract -->> FrontEnd: Display transaction hash
    else failed
        ContractData-->>SupplyChainContract: NOT OK
        SupplyChainContract -->> FrontEnd: Display failure
        deactivate ContractData
    end

    %% Pack Item
    Farmer ->> FrontEnd: Pack Item
    FrontEnd ->> SupplyChainContract: Pack Item
    SupplyChainContract ->> ContractData: Verify ownership and product state
    activate ContractData
    alt successful
        ContractData-->>SupplyChainContract: OK
        SupplyChainContract ->> ContractData: UpdateProduct State
        SupplyChainContract -->> FrontEnd: Display transaction hash
    else failed
        ContractData-->>SupplyChainContract: NOT OK
        SupplyChainContract -->> FrontEnd: Display failure
        deactivate ContractData
    end

    %% Sell Item
    Farmer ->> FrontEnd: Sell Item
    FrontEnd ->> SupplyChainContract: Sell Item
    SupplyChainContract ->> ContractData: Verify ownership and product state
    activate ContractData
    alt successful
        ContractData-->>SupplyChainContract: OK
        SupplyChainContract ->> ContractData: UpdateProduct State and price
        SupplyChainContract -->> FrontEnd: Display transaction hash
    else failed
        ContractData-->>SupplyChainContract: NOT OK
        SupplyChainContract -->> FrontEnd: Display failure
        deactivate ContractData
    end

    %% Buy Item
    Distributer ->> FrontEnd: Buy Item
    activate Distributer
    FrontEnd ->> SupplyChainContract: Buy Item
    SupplyChainContract ->> ContractData: Verify ownership, product state and transfer amount
    activate ContractData
    alt successful
        ContractData-->>SupplyChainContract: OK
        SupplyChainContract ->> ContractData: UpdateProduct State, distributorID and ownerID
        SupplyChainContract -->> FrontEnd: Display transaction hash
        SupplyChainContract ->> Farmer: Transfer Money
        deactivate Farmer
    else failed
        ContractData-->>SupplyChainContract: NOT OK
        SupplyChainContract -->> FrontEnd: Display failure
        deactivate ContractData
    end

    %% Ship Item
    Distributer ->> FrontEnd: Ship Item
    FrontEnd ->> SupplyChainContract: Ship Item
    SupplyChainContract ->> ContractData: Verify ownership and product state
    activate ContractData
    alt successful
        ContractData-->>SupplyChainContract: OK
        SupplyChainContract ->> ContractData: UpdateProduct State
        SupplyChainContract -->> FrontEnd: Display transaction hash
    else failed
        ContractData-->>SupplyChainContract: NOT OK
        SupplyChainContract -->> FrontEnd: Display failure
        deactivate ContractData
    end

    %% Recieve Item
    Retailer ->> FrontEnd: Recieve Item
    activate Retailer
    FrontEnd ->> SupplyChainContract: Recieve Item
    SupplyChainContract ->> ContractData: Verify product state
    activate ContractData
    alt successful
        ContractData-->>SupplyChainContract: OK
        SupplyChainContract ->> ContractData: UpdateProduct State, OwnerID and RetailerID
        SupplyChainContract -->> FrontEnd: Display transaction hash
        deactivate Distributer
    else failed
        ContractData-->>SupplyChainContract: NOT OK
        SupplyChainContract -->> FrontEnd: Display failure
        deactivate ContractData
    end

    %% Purchase Item
    Consumer ->> FrontEnd: Purchase Item
    activate Consumer
    FrontEnd ->> SupplyChainContract: Purchase Item
    SupplyChainContract ->> ContractData: Verify product state
    activate ContractData
    alt successful
        ContractData-->>SupplyChainContract: OK
        SupplyChainContract ->> ContractData: UpdateProduct State, OwnerID and ConsumerID
        SupplyChainContract -->> FrontEnd: Display transaction hash
        deactivate Retailer
    else failed
        ContractData-->>SupplyChainContract: NOT OK
        SupplyChainContract -->> FrontEnd: Display failure
        deactivate ContractData
    end
    
    deactivate Consumer
    deactivate FrontEnd
    deactivate SupplyChainContract


```

### State diagram
```mermaid
stateDiagram-v2
    [*] --> Harvested
    Harvested --> Processed: process [ownerID=msg.sender, state=Harvested]
    Processed --> Packed: pack [ownerID=msg.sender, state=Processed]
    Packed --> ForSale: well [ownerID=msg.sender, state=Packed]
    ForSale --> Sold: buy [ownerID=msg.sender, state=ForSale, productPrice=msg.value]
    Sold --> Shipped: ship [ownerID=msg.sender, state=Sold]
    Shipped --> Received: recieve [state=Shipped]
    Received --> Purchased: purchase [state=Recieved]
    Purchased --> [*]
```

### Libraries
I am using the following libraries:

- **truffle**: Used as development framework.
- **ganache**: Local blockchain used for development.
- **web3**: Used to interact with the ethereum network.
- **mocca**: Used for testing. The developement of this DApp will be test-driven.
### IPFS
The website itself will be hosted on IPFS, making this a truly decentralized service.
