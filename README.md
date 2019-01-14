generate artifactos and crypto
```
configtxgen -profile TwoOrgsOrdererGenesis -channelID syschannel -outputBlock ./channel-artifacts/genesis.block
configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID mychannel
cryptogen generate --config=./crypto-config.yaml
```

import util functions
```
. scripts/utils.sh
```

create channel
```
peer channel create -o orderer0.example.com:7050 -c mychannel -f ./channel-artifacts/channel.tx --tls true --cafile $ORDERER_CA
```

fetch genesisi block (optional)
```
peer channel fetch config mychannel.block -o orderer0.example.com:7050 -c mychannel --tls --cafile $ORDERER_CA
```

join channel
```
for org in 1 2; do for peer in 0 1; do setGlobals $peer $org && peer channel join -b mychannel.block; done; done
```

install chaincode (**use setGlobals to install on both peer0org1 and peer0org2**)
```
setGlobals 0 1
peer chaincode install -n mycc -v 1.0 -l golang -p github.com/chaincode/chaincode_example02/go
setGlobals 0 2
peer chaincode install -n mycc -v 1.0 -l golang -p github.com/chaincode/chaincode_example02/go
```

instantiate chaincode
```
peer chaincode instantiate -o orderer0.example.com:7050 --tls true --cafile $ORDERER_CA -C mychannel -n mycc -l golang -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"
```

invoke chaincode
```
parsePeerConnectionParameters 0 1 0 2
```
```
peer chaincode invoke -o orderer0.example.com:7050 --tls true --cafile $ORDERER_CA -C mychannel -n mycc $PEER_CONN_PARMS -c '{"Args":["invoke","a","b","10"]}'
```

query chaincode
```
peer chaincode query -C mychannel -n mycc -c '{"Args":["query","b"]}'
```

getch config block on syschannel
```
peer channel fetch config configBlock.pb -o orderer0.example.com:7050 -c syschannel --tls --cafile $ORDERER_CA
```

start orderer3
```
docker-compose up orderer3.example.com
```

set env var before invoking
```
setGlobals 0 1
```
