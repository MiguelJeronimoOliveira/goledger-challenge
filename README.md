# FABCAR Chaincode Enhancements & GoLedger Setup

## Table of Contents

- [FABCAR Chaincode Enhancements](#fabcar-chaincode-enhancements)
  - [Car struct](#car-struct)
  - [createCar method](#createcar-method)
  - [queryCarHistory method](#querycarhistory-method)
  - [changeCarColor method](#changecarcolor-method)
  - [deleteCar method](#deletecar-method)
- [GoLedger Challenge](#goledger-challenge)
  - [Install the prerequisites](#install-the-prerequisites)
  - [Set up the network](#set-up-the-network)
- [FABCAR Chaincode](#fabcar-chaincode)
  - [Initialize ledger](#initialize-ledger)

---

## FABCAR Chaincode Enhancements

The `fabcar.go` file has been updated to include an optional `"Year"` attribute in the `Car` struct. It's now possible to provide the car's manufacturing year using the `createCar` method.

### Car struct

The `Car` struct now includes the `Year` field:

```go
type Car struct {
	Make   string `json:"make"`
	Model  string `json:"model"`
	Colour string `json:"colour"`
	Owner  string `json:"owner"`
	Year   int    `json:"year"` // New attribute
}
```

### createCar method

The `createCar` method now expects 6 arguments. The sixth argument (`args[5]`) sets the car's year.

- There is a validation to ensure the `Year` value is numeric. If not, it returns the error: `"Year must be a numeric value"`.

```go
if len(args) != 6 {
	return shim.Error("Incorrect number of arguments. Expecting 6")
}

year, err := strconv.Atoi(args[5])
if err != nil {
	return shim.Error("Year must be a numeric value")
}
```

### queryCarHistory method

- Created to show a car's transaction history.
- Takes one argument (`args[0]`), the car's key (e.g., `"CAR0"`).
- Returns: `TxId`, `Value`, `Timestamp`, `IsDelete`.

Query example:

```bash
peer chaincode query -C mychannel -n fabcar -c '{"Args":["queryCarHistory", "CAR0"]}'
```

### changeCarColor method

- Created to change a car's color.
- Takes 2 arguments:
  - `args[0]`: the car key (e.g., `"CAR0"`)
  - `args[1]`: the new color (e.g., `"Red"`)

Invoke example:

```bash
peer chaincode invoke -o orderer.example.com:7050 -C mychannel -n fabcar -c '{"function":"changeCarColor","Args":["CAR0", "Red"]}' --waitForEvent --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/t...
```

### deleteCar method

- Created to delete a car from the ledger.
- Takes 1 argument (`args[0]`), the car key (e.g., `"CAR0"`).
- The deletion remains in the transaction history and can be audited using `queryCarHistory`.

Invoke example:

```bash
peer chaincode invoke -o orderer.example.com:7050 -C mychannel -n fabcar -c '{"function":"deleteCar","Args":["CAR0"]}' --waitForEvent --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```

---

## GoLedger Challenge

Start your own permissioned blockchain using Hyperledger Fabric. Set up the network, create a channel, join a peer, and install and instantiate a smart contract (chaincode).

Recommended environment: UNIX-like system, cURL, Docker, Go.

### Install the prerequisites

- [cURL](https://curl.haxx.se/download.html)
- [Docker + Compose](https://www.docker.com/)
- [Go](https://golang.org/dl/)
- Install Fabric binaries and Docker images:

```bash
curl -sSL http://bit.ly/2ysbOFE | bash -s -- 2.5.12 2.5.12 1.5.15
```

Make sure the Fabric binaries are in your `$PATH`.

### Set up the network

#### Generate certificates and create channel artifacts

```bash
cd /goledger-network
./bgldgr.sh generate
```

#### Bring up the network containers

```bash
docker-compose -f docker-compose-cli.yaml up -d
```

#### Create and Join Channel

```bash
docker exec -it cli bash

peer channel create -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

peer channel join -b mychannel.block
```

#### Install and Instantiate Chaincode

```bash
cd /chaincode/fabcar/go
go mod vendor

cd /opt/gopath/src/github.com/chaincode/fabcar/go
peer lifecycle chaincode package fabcar<VERSION>.tar.gz --path github.com/chaincode/fabcar/go/ --lang golang --label fabcar_<VERSION>
peer lifecycle chaincode install fabcar<VERSION>.tar.gz
```

Approve and commit the chaincode as described (replace `<VERSION>` and `<PACKAGE_ID>` accordingly).

---

## FABCAR Chaincode

FABCAR is a sample chaincode provided by Hyperledger to manage car registration, including attributes like make, model, color, year, and owner.

### Initialize ledger

Initialize the ledger with sample records:

```bash
peer chaincode invoke -o orderer.example.com:7050 -C mychannel -n fabcar -c '{"function":"initLedger","Args":[]}' --waitForEvent --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```

List the registered cars:

```bash
peer chaincode query -C mychannel -n fabcar -c '{"Args":["queryAllCars"]}'
```