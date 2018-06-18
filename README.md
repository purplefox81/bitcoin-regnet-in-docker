# bitcoin-testnet-box
[![docker pulls](https://img.shields.io/docker/pulls/freewil/bitcoin-testnet-box.svg?style=flat)](https://hub.docker.com/r/freewil/bitcoin-testnet-box/)

## Steps to Build Everything from Scratch (by Yang ChenMing)

1. make sure docker is installed, this repo is cloned to your local disk
2. bitcoin-testnet-box> docker build -t bitcoin-testnet-box .
3. docker run -t -i -p 19001:19001 -p 19011:19011 --name bitcoin-testnet bitcoin-testnet-box
4. Now we are in the command prompt tester@75a8e858fcbb:~/bitcoin-testnet-box$
5. Start two nodes
  * make start
6. Verify the nodes are up and running 
  * bitcoin-cli -datadir=1 getnetworkinfo (or getinfo)
  * bitcoin-cli -datadir=2 getnetworkinfo (or getinfo)
7. Generate some transaction (and enough of them so that there will be a balance in the first wallet)
  * make generate BLOCKS=100
8. Verify the transactions are recorded in blocks and values are reflected in wallets
  * bitcoin-cli -datadir=1 getwalletinfo (look for 'xxx_balance' fields)
  * bitcoin-cli -datadir=1 getblockchaininfo (look for 'blocks' field)
9. Generate a new address for node2 and tranfer some coin to it
  * make address2 
  * make sendfrom1 ADDRESS=... AMOUNT=5 (Address is taken from the printed adress from the previous command)
  * bitcoin-cli -datadir=1 getwalletinfo
  * bitcoin-cli -datadir=2 getwalletinfo (note the 'unconfirmed_balance' field)
  * make generate (we generate one more block so that the previous transfer will be confirmed)
  * bitcoin-cli -datadir=2 getwalletinfo (look for the 'balance' field)
9. Stop and restore to the original state
  * make stop
  * make clean


Sample Content for Makefile

BITCOIND=bitcoind                                                 
BITCOINGUI=bitcoin-qt                                             
BITCOINCLI=bitcoin-cli                                            
B1_FLAGS=                                                         
B2_FLAGS=                                                         
B1=-datadir=1 $(B1_FLAGS)                                         
B2=-datadir=2 $(B2_FLAGS)                                         
BLOCKS=1                                                          
ADDRESS=                                                          
AMOUNT=                                                           
ACCOUNT=                                                          
                                                                  
start:                                                            
        $(BITCOIND) $(B1) -daemon                                 
        $(BITCOIND) $(B2) -daemon                                 
                                                                  
start-gui:                                                        
        $(BITCOINGUI) $(B1) &                                     
        $(BITCOINGUI) $(B2) &                                     
                                                                  
generate:                                                         
        $(BITCOINCLI) $(B1) generate $(BLOCKS)                    
                                                                  
getinfo:                                                          
        $(BITCOINCLI) $(B1) getinfo                               
        $(BITCOINCLI) $(B2) getinfo                               
                                                                  
sendfrom1:                                                        
        $(BITCOINCLI) $(B1) sendtoaddress $(ADDRESS) $(AMOUNT)    
                                                                  
sendfrom2:                                                        
        $(BITCOINCLI) $(B2) sendtoaddress $(ADDRESS) $(AMOUNT)    
                                                                  
address1:                                                         
        $(BITCOINCLI) $(B1) getnewaddress $(ACCOUNT)              
                                                                  
address2:                                                         
        $(BITCOINCLI) $(B2) getnewaddress $(ACCOUNT)              
                                                                  
stop:                                                             
        $(BITCOINCLI) $(B1) stop                                  
        $(BITCOINCLI) $(B2) stop                                  
                                                                  
clean:                                                            
        find 1/regtest/* -not -name 'server.*' -delete            
        find 2/regtest/* -not -name 'server.*' -delete            



===== ORIGINAL README.md ======


Create your own private bitcoin testnet

You must have `bitcoind` and `bitcoin-cli` installed on your system and in the
path unless running this within a [Docker](https://www.docker.com) container
(see [below](#using-with-docker)).

## Starting the testnet-box

This will start up two nodes using the two datadirs `1` and `2`. They
will only connect to each other in order to remain an isolated private testnet.
Two nodes are provided, as one is used to generate blocks and it's balance
will be increased as this occurs (imitating a miner). You may want a second node
where this behavior is not observed.

Node `1` will listen on port `19000`, allowing node `2` to connect to it.

Node `1` will listen on port `19001` and node `2` will listen on port `19011`
for the JSON-RPC server.


```
$ make start
```

## Check the status of the nodes

```
$ make getinfo
bitcoin-cli -datadir=1  getinfo
{
    "version" : 90300,
    "protocolversion" : 70002,
    "walletversion" : 60000,
    "balance" : 0.00000000,
    "blocks" : 0,
    "timeoffset" : 0,
    "connections" : 1,
    "proxy" : "",
    "difficulty" : 0.00000000,
    "testnet" : false,
    "keypoololdest" : 1413617762,
    "keypoolsize" : 101,
    "paytxfee" : 0.00000000,
    "relayfee" : 0.00001000,
    "errors" : ""
}
bitcoin-cli -datadir=2  getinfo
{
    "version" : 90300,
    "protocolversion" : 70002,
    "walletversion" : 60000,
    "balance" : 0.00000000,
    "blocks" : 0,
    "timeoffset" : 0,
    "connections" : 1,
    "proxy" : "",
    "difficulty" : 0.00000000,
    "testnet" : false,
    "keypoololdest" : 1413617762,
    "keypoolsize" : 101,
    "paytxfee" : 0.00000000,
    "relayfee" : 0.00001000,
    "errors" : ""
}
```

## Generating blocks

Normally on the live, real, bitcoin network, blocks are generated, on average,
every 10 minutes. Since this testnet-in-box uses Bitcoin Core's (bitcoind)
regtest mode, we are able to generate a block on a private network
instantly using a simple command.

To generate a block:

```
$ make generate
```

To generate more than 1 block:

```
$ make generate BLOCKS=10
```

## Need to generate at least 100 blocks before there will be a balance in the first wallet
```
$ make generate BLOCKS=200
```

## Verify that there is a balance on the first wallet
```
$ make getinfo
```

## Generate a wallet address for the second wallet
```
$ make address2
```

## Sending bitcoins
To send bitcoins that you've generated to the second wallet: (be sure to change the ADDRESS value below to wallet address generated in the prior command)

```
$ make sendfrom1 ADDRESS=mxwPtt399zVrR62ebkTWL4zbnV1ASdZBQr AMOUNT=10
```

## Does the balance show up?
Run the getinfo command again. Does the balance show up? Why not?
```
$ make getinfo
```

## Generate another block
```
$ make generate
```

## Stopping the testnet-box

```
$ make stop
```

To clean up any files created while running the testnet and restore to the
original state:

```
$ make clean
```

## Using with docker
This testnet-box can be used with [Docker](https://www.docker.com/) to run it in
an isolated container.

### Building docker image

Pull the image
  * `docker pull freewil/bitcoin-testnet-box`

or build it yourself from this directory
  * `docker build -t bitcoin-testnet-box .`

### Running docker container
The docker image will run two bitcoin nodes in the background and is meant to be
attached to allow you to type in commands. The image also exposes
the two JSON-RPC ports from the nodes if you want to be able to access them
from outside the container.
      
   `$ docker run -t -i -p 19001:19001 -p 19011:19011 freewil/bitcoin-testnet-box`

or if you built the docker image yourself:

   `$ docker run -t -i -p 19001:19001 -p 19011:19011 bitcoin-testnet-box`

