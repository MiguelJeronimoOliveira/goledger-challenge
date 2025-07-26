## FABCAR Chaincode Enhancements

The `fabcar.go` chaincode has been updated to include an optional "Year" attribute in the `Car` struct, and it's now possible to provide the car's year at creation using the `createCar` method.

### `Car` struct

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

### `createCar` method

The `createCar` method now includes an argument to persist the `Year` field whith your validations
-	The `createCar` function now expects 6 arguments. The sixth argument (args[5]) is used to set the Year.
-	There is a validation in place to ensure that the `Year` value provided is a numeric value. If it's not, an error "Year must be a numeric value" is returned.

```go
if len(args) != 6 {
		return shim.Error("Incorrect number of arguments. Expecting 6")
	}

	year, err := strconv.Atoi(args[5])

	if err != nil {
		return shim.Error("Year must be a numeric value")
	}
```

### `queryCarHistory` method
-	Created to show the transaction  history of a car
-	It receives 1 argument (args[0]) which is indeed the car key (Ex: `"CAR0"`)
-	The method shows information about transaction history changes, including
	- `TxId` (Transaction ID) 
	- `Value` (Transaction Value)
	- `Timestamp` (Transaction timeStamp)
	- `IsDelete` (If the transaction resulted in a deletion)

 you can run the following command (inside the cli container):

	peer chaincode query -C mychannel -n fabcar -c '{"Args":["queryCarHistory", "CAR0"]}'

### `changeCarColor` method
-	Created to change the car`s color
-	it receives 2 arguments similar to the `changeCarOwner` method
	- The First argument (args[0]) is indeed the car key (Ex: `"CAR0"`)
	- The Second Argument (args[1]) is the new color you want to set (Ex `"Red"`)

To invoke the transaction you can run the following command (inside the cli container):

	peer chaincode invoke -o orderer.example.com:7050 -C mychannel -n fabcar -c '{"function":"changeCarColor","Args":["CAR0", "Red"]}' --waitForEvent --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt

### `deleteCar` method
-	Created to delete a car
-	it receives 1 argument (args[0]) which is indeed the car key (Ex: `"CAR0"`)
- 	The method deletes the car from the ledger. You can still view its transaction history, including the deletion event, using the queryCarHistory method with the corresponding car key.

To invoke the transaction you can run the following command (inside the cli container):

	peer chaincode invoke -o orderer.example.com:7050 -C mychannel -n fabcar -c '{"function":"deleteCar","Args":["CAR0"]}' --waitForEvent --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt



# GoLedger Challenge

Start your own permissioned blockchain. To do this, you will set up a network, create a channel, join a peer to that channel, install and instantiate a smart contract (also known as chaincode in Hyperledger Fabric).

We recommend using a UNIX-like machine (Linux/macOS) for this. Additionally, you'll need to install cURL and Docker.

Note: Whenever you want to start from scratch, make sure to clear your Docker containers and volumes.

## Install the prerequisites

- Install cURL (https://curl.haxx.se/download.html)
- Install Docker and Docker Compose (https://www.docker.com/)
- Install Go (https://golang.org/dl/)
- Download the Hyperledger Docker images

        curl -sSL http://bit.ly/2ysbOFE | bash -s -- 2.5.12 2.5.12 1.5.15
    - Make sure the Fabric binaries are in your `$PATH`.
	- Note: The `bgldgr.sh` script actually searches for the binaries in the `/fabric-samples/bin` directory.

## Set up the network
### Generate certificates and create channel artifacts

Navigate to the `/goledger-network` folder. Now, we need to generate the certificates used by the peer and orderer so they can sign and validate transactions.

    ./bgldgr.sh generate

This command will create the necessary certificates and channel artifacts.

### Bring up the network containers

Assuming you have Docker and Docker Compose installed, the following command will start the network components.

    docker-compose -f docker-compose-cli.yaml up -d

You should see the following output:

    Creating orderer.example.com    ... done
    Creating peer0.org1.example.com ... done
    Creating cli                    ... done

### Create and Join Channel

When deploying a local network, you can use the `cli` container to perform operations on the network. To do this, you need to enter the container. Use the following command:

    docker exec -it cli bash

Create the channel.

	peer channel create \
    -o orderer.example.com:7050 \
    -c mychannel \
    -f ./channel-artifacts/channel.tx \
    --tls \
    --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

Join the channel.

    peer channel join -b mychannel.block

### Install and Instantiate Chaincode

In the `/chaincode/fabcar/go` folder, download the chaincode package dependencies.

    go mod vendor

Inside the `cli` container, navigate to `/opt/gopath/src/github.com/chaincode/fabcar/go`.

    cd /opt/gopath/src/github.com/chaincode/fabcar/go

Package the chaincode. Replace `<VERSION>` with the version you want to package (Ex, `1.0` for the first version, `2.0` for the second, and so on).

    peer lifecycle chaincode package fabcar<VERSION>.tar.gz \
		--path github.com/chaincode/fabcar/go/ \ 
		--lang golang \ 
	    --label fabcar_<VERSION>

With the `fabcar<VERSION>.tar.gz` package created, install the chaincode.

    peer lifecycle chaincode install fabcar<VERSION>.tar.gz

From the previous step, copy and save the installed chaincode's Package ID (Ex, `fabcar_1.0:93c9df9bf47532689ed6d78654787557740439fdd7435e06b5dcc63e07fe3cea`).

Now, approve the chaincode definition for the organization. Replace `<PACKAGE_ID>` in the command with the previously saved value.

    peer lifecycle chaincode approveformyorg \
	-o orderer.example.com:7050 \
	--ordererTLSHostnameOverride orderer.example.com \
	--channelID mychannel \
	--name fabcar \
	--version <VERSION> \
	--package-id <PACKAGE_ID> \
	--sequence <VERSION> \
	--tls \
	--cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

Check the chaincode commit readiness. The approval value should be `true`.

	peer lifecycle chaincode checkcommitreadiness \
		--channelID mychannel \
		--name fabcar \
		--version <VERSION> \
		--sequence <VERSION> \
		--tls \
		--cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem \
		--output json

Commit the chaincode definition to the channel.

	peer lifecycle chaincode commit \
		-o orderer.example.com:7050 \
		--ordererTLSHostnameOverride orderer.example.com \
		--channelID mychannel \
		--name fabcar \
		--version <VERSION> \
		--sequence <VERSION> \
		--tls \
		--cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem \
		--peerAddresses peer0.org1.example.com:7051 \
		--tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt

Verify the chaincode has been successfully committed to the channel.

    peer lifecycle chaincode querycommitted --channelID mychannel --name fabcar

## FABCAR Chaincode

The chaincode you just instantiated on your newly created channel is called FABCAR and is one of the sample chaincodes provided by Hyperledger. This chaincode handles car registration and management, including attributes like (make, model, color, year, and owner). The source code is located at `/chaincode/fabcar/go` in the repository root folder.

### Initialize ledger

Your chaincode is freshly instantiated, so let’s initialize the ledger with some cars. The FABCAR chaincode provides an operation to do this easily. To invoke a transaction, you can run the following command (inside the `cli` container):

    peer chaincode invoke -o orderer.example.com:7050 -C mychannel -n fabcar -c '{"function":"initLedger","Args":[]}' --waitForEvent --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt

Now check the cars on the ledger:

    peer chaincode query -C mychannel -n fabcar -c '{"Args":["queryAllCars"]}'

