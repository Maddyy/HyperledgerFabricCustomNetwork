# HyperledgerFabricCustomNetwork
Build Hyperledger Fabric Custom Network without Using BYFN script !!


##### Generate certificates using cryptogen tool #########
/home/hyper/Desktop/hyperledger/golang/src/fabric1.4/fabric-samples/bin/cryptogen generate --config=/home/hyper/Desktop/hyperledger/golang/src/fabric1.4/fabric-samples/medical-authority/crypto-config/crypto-config.yaml

export SYS_CHANNEL=ima-sys-channel
"#########  Generating Orderer Genesis block ##############"

/home/hyper/Desktop/hyperledger/golang/src/fabric1.4/fabric-samples/bin/configtxgen -profile TwoOrgsOrdererGenesis -channelID "ima-sys-channel" -outputBlock /home/hyper/Desktop/hyperledger/golang/src/fabric1.4/fabric-samples/medical-authority/channel-artifacts/genesis.block

### Generating channel configuration transaction 'channel.tx' ###

/home/hyper/Desktop/hyperledger/golang/src/fabric1.4/fabric-samples/bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx /home/hyper/Desktop/hyperledger/golang/src/fabric1.4/fabric-samples/medical-authority/channel-artifacts/channel.tx -channelID "imamedicalchannel"

"#######    Generating anchor peer update for Org1MSP   ##########

/home/hyper/Desktop/hyperledger/golang/src/fabric1.4/fabric-samples/bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate /home/hyper/Desktop/hyperledger/golang/src/fabric1.4/fabric-samples/medical-authority/channel-artifacts/HospitalMSPanchors.tx -channelID "imamedicalchannel" -asOrg HospitalMSP
 
#######    Generating anchor peer update for Org2MSP   ##########
/home/hyper/Desktop/hyperledger/golang/src/fabric1.4/fabric-samples/bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate /home/hyper/Desktop/hyperledger/golang/src/fabric1.4/fabric-samples/medical-authority/channel-artifacts/InsurancecompanyMSPanchors.tx -channelID "imamedicalchannel" -asOrg InsurancecompanyMSP
###############################################################################################################

export IMAGE_TAG="latest"
export SYS_CHANNEL="ima-sys-channel"
docker-compose  -f docker-compose-cli.yaml -f docker-compose-ca.yaml -f docker-compose-couch.yaml up -d
############################################################################################################
docker exec -it cli bash
############################################################################################################
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/hospital.sharad.com/users/Admin@hospital.sharad.com/msp
CORE_PEER_ADDRESS=aiimsindia.hospital.sharad.com:7051
CORE_PEER_LOCALMSPID="HospitalMSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/hospital.sharad.com/peers/aiimsindia.hospital.sharad.com/tls/ca.crt
#############################################################################################################
peer channel create -o ministryofhealth.sharad.com:7050 -c "imamedicalchannel" -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/sharad.com/orderers/ministryofhealth.sharad.com/msp/tlscacerts/tlsca.sharad.com-cert.pem
#############################################################################################################
peer channel update -o ministryofhealth.sharad.com:7050 -c "imamedicalchannel" -f ./channel-artifacts/HospitalMSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/sharad.com/orderers/ministryofhealth.sharad.com/msp/tlscacerts/tlsca.sharad.com-cert.pem
#############################################################################################################

peer channel join -b imamedicalchannel.block
#############################################################################################################

CORE_PEER_ADDRESS=aiimsindia.hospital.sharad.com:7051 peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/

CORE_PEER_ADDRESS=aiimsindia.hospital.sharad.com:7051 peer chaincode instantiate -o ministryofhealth.sharad.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/sharad.com/orderers/ministryofhealth.sharad.com/msp/tlscacerts/tlsca.sharad.com-cert.pem -C "imamedicalchannel" -n mycc -l golang -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "OR ('HospitalMSP.peer','InsurancecompanyMSP.peer')"
###########################################Open New Terminal###################################################
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/hospital.sharad.com/users/Admin@hospital.sharad.com/msp
CORE_PEER_ADDRESS=uhgusa.hospital.sharad.com:8051
CORE_PEER_LOCALMSPID="HospitalMSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/hospital.sharad.com/peers/uhgusa.hospital.sharad.com/tls/ca.crt
#############################################################################################################
peer channel join -b imamedicalchannel.block
#############################################################################################################
CORE_PEER_ADDRESS=uhgusa.hospital.sharad.com:8051 peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/
#############################################################################################################

CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/insurancecompany.sharad.com/users/Admin@insurancecompany.sharad.com/msp
CORE_PEER_ADDRESS=sbi.insurancecompany.sharad.com:9051
CORE_PEER_LOCALMSPID="InsurancecompanyMSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/insurancecompany.sharad.com/peers/sbi.insurancecompany.sharad.com/tls/ca.crt
#############################################################################################################
peer channel join -b imamedicalchannel.block
#############################################################################################################
CORE_PEER_ADDRESS=sbi.insurancecompany.sharad.com:9051 peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/
#############################################################################################################
CORE_PEER_ADDRESS=sbi.insurancecompany.sharad.com:9051 peer chaincode instantiate -o ministryofhealth.sharad.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/sharad.com/orderers/ministryofhealth.sharad.com/msp/tlscacerts/tlsca.sharad.com-cert.pem -C "imamedicalchannel" -n mycc -l golang -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "AND ('HospitalMSP.peer','InsurancecompanyMSP.peer')"
#############################################################################################################

CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/insurancecompany.sharad.com/users/Admin@insurancecompany.sharad.com/msp
CORE_PEER_ADDRESS=uhg.insurancecompany.sharad.com:10051
CORE_PEER_LOCALMSPID="InsurancecompanyMSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/insurancecompany.sharad.com/peers/uhg.insurancecompany.sharad.com/tls/ca.crt
#############################################################################################################
peer channel join -b imamedicalchannel.block
#############################################################################################################

CORE_PEER_ADDRESS=uhg.insurancecompany.sharad.com:10051 peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/
#############################################################################################################
peer chaincode invoke -o ministryofhealth.sharad.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/sharad.com/orderers/ministryofhealth.sharad.com/msp/tlscacerts/tlsca.sharad.com-cert.pem -C "imamedicalchannel" -n mycc --peerAddresses aiimsindia.hospital.sharad.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/hospital.sharad.com/peers/aiimsindia.hospital.sharad.com/tls/ca.crt --peerAddresses sbi.insurancecompany.sharad.com:9051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/insurancecompany.sharad.com/peers/sbi.insurancecompany.sharad.com/tls/ca.crt -c '{"Args":["invoke","a","b","10"]}' 
#############################################################################################################
peer chaincode query -C imamedicalchannel -n mycc -c '{"Args":["query","a"]}'
#############################################################################################################
