# cs686_BlockChain_P5 - Lending system (such as crowd funding platform)

# Application has following features
1. Client can 'ask' for a sum of tokens.
2. This 'ask' request is share among newtork users
3. Clients can choose a 'ask' request and 'promise' and make a promise for a sum of money.
4. All previous promises for the specific 'ask' request are tracked, and if the sum of promises mount to 'asked' tokens, then money is transferred for appropriate users.

Additionally, clients can also use peer to peer token transfer.

Available balance is different from Actual balance. Available balance takes into account the sum of tokens already promised.
Available balance is used for checking validity of transaction.


# (1) Crypto 
Achieving data integrity by use of Signature.

## security.go
Implements the public-private keys and Hash generation. Also creating signature and verifying signature.
Contains two data structs - Identity and PublicIdentity.
GetPublicIdentity stuct in security.go - for Node to be able to get its PublicId in new variable.

Using golang rsa PKCS1v15 to sign and verify.

1. new data structs in security.go
  - Identity {
      privateKey,
      PublicKey,
      Label,
      }
  
  - PublicIdentity {
      PublicKey,
      Label,
      }

## peerlist.go
 ### Todo - merge logic of processing PeerMap and PeerMapPid 
 PeerList - new variables in struct
  secureId    - added to contain secureId of dataType Identity.
  peerMapPid  - now also contains PeerMapPid
  PeerMapPid  - a Map of - Addr of Node (key) - and PublicIdentity (value) of peers.
  
  Methods added - 
  InjectPeerMapPidJson method - to inject receieved pidJson in receiever map
  And other methods relating PeerMapPid, parallel in logic with PeerMap
 
## heartbeat.go
 HeartBeat changed to now also include
   Pid
   SignForBlockJson - for BlockJson
   PeerMapPidJson  

## handlers.go
(Todo - sender have to encrypt the heartbeat with public key of receving peers, Receiever have to decrypt the heartbeat with private key of itself. - only when confidentiality is needed)

1) In StartHeartBeat func - AND - In SendBlockBeat func -
Add new params to PrepareHeartBeatData func call
    add signature for blockjson
    add Pid of Sender
    
(Add Encrypt heartbeat with public key of "to whom the heartbeat is being sent". (in for loop))
((2)In HeartBeatReceive func -)
(Add functionality to Decrypt heartbeat as soon as receieved)



## API
GET /uploadpids - to download the public key map from Register server to be used first time along with download blockchain.

HeartBeat send and Receieve now additionally deals with Signature of sender and its verification by receiever.

(Have to add functionality to send encrypted blockjson and decrypt blockjson - in handler.go (funcs available in security.go))


# (2) Currency
Adding mechanism to enable comodity(digital token) to be exchanged between peers.

Peers have 1000 in begining by default for now.
They can create tx, the tx then goes to tx pool.
Txs are picked by a peer from Tx pool.
Tx remain in pool until it is part of canonical chain

##### More on transactions
There are 2 type of transaction
- Borrowing Tx
- Lending Tx
Generics ?? <<<<<<<<<<<< to understand type of Transaction
------------------------- some ALGO ---------------------
For every Borrowing TX 
  Create a Promised Struct {  PromiseMPT<Lender TX, Lending Amount> } 
And add it in PromiseList struct { Map < BorrowingTX, Promised Struct > }

if Borrowing Tx 
  Create a Promised Struct {  PromiseMPT<Lender TX, Lending Amount> } 
  And add it in PromiseList { Map < BorrowingTX, Promised Struct > }
else if Lending Tx
  loop over PromiseList to find the matching BorrowingTXId THEN
    >if lender id is new 
      Add lender id, lended amount on Promised MPT for that Borrowing Tx in the PromiseList Map
    else if lender id is old
      Get that lender id, increase the existing amout by this new lended tokens
    >if Total Promised Amt meets the Borrowing Requirement
      Process the entry in PromiseList Map for that Borrowing TX !! processPromises(Borrowing Tx) !!(1)
        
!!(1)!!
processPromises(Borrowing Tx)
  get Entry from PromiseList struct { Map < BorrowingTX, Promised Struct > } for the corresponding Borrowing Tx
  Add the total of lended tokens 
  Remove the entry from PromiseList Map
  
---------------------------------------------------------
------------------------- some ALGO ---------------------
When Node receieves heartbeat :
  ...
  gets mpt 



---------------------------------------------------------
## transaction.go
A transaction is considered valid if tokens needed(including fees) >= balance in Book - Promised

Data structs in transaction.go
  1) Transaction<type of Tx - Borrowing or Lending ?> {
    Id - is hash of tx <<<<<<<<<
    From
    To
    AmountOfTokens <<<<<<<<<<<<<
    Timestamp
  }

  2) TransactionPool {
     list of transaction
  }

  3) TransactionBeat {
      Transaction
      FromPid
      TxSignature  <<<<<<<<
  }

Funcs ->
CreateTransaction func -> takes params From public Id, To public Id, Tokens and Timestamp and -> returns Tx.
NewTransactionBeat func -> takes params Tx, From public Id and FromSig and -> returns TransactionBeat.
CreateTransactionBeat func ->  takes params Tx and Identity and -> returns. 
AddToTransactionPool func -> takes a transaction and adds it to TransactionPool.
DeleteFromTransactionPool func -> takes transaction id and deletes it from TransactionPool.
ReadFromTransactionPool func -> takes in no. of tx to read and returns txmap of as many txs.




## wallet.go
Data structs in wallet.go
  - Wallet {
    Balance, - map[string]float64
    mutex,
  }
  where Balance is map of unit and currency amt.
  
## balanceBook.go
Data structs in balanceBook.go
  - BalanceBook {
    Book,     - mpt
    Promised, - mpt
    mutex,
  }


Funcs ->
UpdateBalanceBook
GetBalance

//todo 
_______________________________________________________________ IsBalanceEnough finc
IsBalanceEnough() {
 - takes in key and needed balance - and returns true of false based on (Book and Promised)
}
_______________________________________________________________   generate balancebook and Promise book for a Chain
GenerateBalanceAndPromise(SBC SyncBlockchain) {
  - use function in canonical chain to get - the canonical blockchain then
  - start reading from 1st height block and read all the transactions to build up balancebook and promise book
}

_______________________________________________________________ Reading all transaction of one block 
 - get a Block 
 - convert block to key value pairs
 - for every key value pairs - check and update ... balancebook and promise

todo //

## handlers.go
Add data structs to Keep Balance, 

Need encryption of Holding account ????
if for every borrowing tx - a holding account is created.
And Lending amt is kept there until it is ready for use by borrowers.

## API

GET /ShowWallet

GET /ShowBalanceBook

GET /ShowTransactionPool

POST /transaction
  req body should contain a Transaction

POST /transactionBeatRecv
  req body should contain TransactionBeat( {Tx, FromPid, TxSignature, Hops} ) 


========================================================================================
========================================================================================
=====================================Client=============================================
========================================================================================
========================================================================================

starts normal but do not call the start api
Instead calls GET /cover api - which shows the starting page for client - func startClient()
then client can signup to get a key pair 
then store it.
After that client cal login by pasting the whole thing as -key- and -addr- and then send to any miner (for now 6686)
The miner will verify if it - if verified send the peerlist to the client. - and show Client Page (where client interacts)

data structres a Client will have ->
  - ClientId
  - SELF_ADDR from Init function
  - use here Peers of type PeerList
  



============================
two miners - tx beat  - 2 conditions
1. miner new - ask for txPool
2. miner old - real time communicate new transaction

when 2 miners exist - client will have two miners in bcHolders

------
keyValuePairs := p1.MerklePatriciaTrie(b.Block(uBlock).Value).GetAllKeyValuePairs()
------

when somebody signup - miner will initiate a default tx of 1000

--> . Have to test - when sender sends more than they have
--> . Have to Add Towards Txid - promise requirement
--> . Now adding miner in Block and Fees

Arrray - 

