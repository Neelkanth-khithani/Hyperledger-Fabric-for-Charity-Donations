# Hyperledger Fabric for Charity Donations

## Project Overview

This case study documents the development and deployment of a **permissioned blockchain solution** aimed at enhancing **transparency and accountability** in charity donations. The system provides an **immutable, shared ledger** viewable by Donors, NGOs, and Auditors, ensuring an auditable record of funds from initial contribution to final allocation. The entire solution was built and validated within a local development environment.

![architecture-diagram](/images/architecture_diagram.png)

## Technologies & Environment

The following is a concise summary of the critical technologies, tools, and versions used to build and operate the Hyperledger Fabric network:

| Category | Technology/Tool | Version | Primary Role |
| :--- | :--- | :--- | :--- |
| **Blockchain** | **Hyperledger Fabric** | v2.2.9 (Binaries) | The foundational permissioned blockchain framework for the multi-organizational network. |
| **Smart Contract Logic** | **Go (Golang)** | v1.17.6 | The programming language used to develop the core business logic (Chaincode). |
| **Development Runtime** | **IBM Microfab** | Latest (Containerized) | Tool used to rapidly bootstrap and manage the complete Fabric network (Peers, Orderer, CAs). |
| **Containerization** | **Docker Engine** | Latest Installed | Used to host the Microfab environment and ensure configuration consistency. |
| **Host Environment** | **Windows 11 (WSL 2)** | Ubuntu 20.04.6 LTS | Operating system used to execute all setup, deployment, and testing commands. |
| **Client Interface** | **Hyperledger Fabric `peer` CLI** | v2.2.9 | Used for all chaincode lifecycle management (install, approve, commit) and transaction execution. |
| **Utility Tooling** | **Weft (`@hyperledgendary/weftility`)** | Global NPM Install | Used to extract network cryptographic material (wallets, gateways, MSPs) from Microfab. |

# Table of Contents

- [Step 1: Installations](#step-1-installations)
  - [Docker](#docker)
  - [cURL](#curl)
  - [Node.js (v18)](#nodejs-v18)
  - [Go (Golang)](#go-golang)
  - [Docker Compose](#docker-compose)
  - [Weft](#weft)
- [Step 2: Bootstrapping the Network using IBM Microfab](#step-2-bootstrapping-the-network-using-ibm-microfab)
  - [Run the Network on Docker](#on-terminal-1-run-the-network-on-docker)
  - [Setup Environment Variables and Binaries](#on-terminal-2-run-the-project-to-setup-the-environment-variables-and-binaries)
  - [Create the HFN-Charity-Contract Project](#create-the-hfn-charity-contract-project-to-write-smart-contract)
  - [Deploy the Chaincode](#on-terminal-2-deploy-the-chaincode)
  - [Chaincode Lifecycle and Deployment (DonorOrg)](#chaincode-lifecycle-and-deployment-donororg)
- [Step 3: Transaction Execution and Testing](#step-3-transaction-execution-and-testing)

# Step 1: Installations

- Update and Upgrade System Packages
    - `sudo apt update -y`
    - `sudo apt upgrade -y`

## Docker

1. Install Docker engine
    -  `sudo apt install docker.io -y`
2. Start the Docker service
    - `sudo service docker start`
3. Add current user to Docker group to run Docker without sudo
    - `sudo usermod -aG docker $USER`
4. Apply new group membership without logout/login
    - `newgrp docker`
    -  `exit`
5. Test Docker installation by running a sample container
    - `docker run hello-world`

![docker-run](/images/docker-run.png)

6. Display installed Docker version
    - `docker --version`

![docker-version](/images/docker-version.png)

## cURL
It is used here to securely download the official NodeSource setup script.

1. Install curl utility
    - `sudo apt-get install curl`

![install-curl](/images/install-curl.png)

2. Verify Curl Installation
    - `curl --version`

![curl-version](/images/curl-version.png)


## Node.js (v18)
Required to install utility tools and to run Hyperledger Fabric client applications built using the Fabric Gateway SDK.

1. Fetch the Node.js v18 setup script
    - `curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -`

![install-node-1](/images/install-node-1.png)

2. Install the Node.js runtime and npm
    - `sudo apt install -y nodejs`

![install-node-2](/images/install-node-2.png)

3. Checks the versions of both the Node.js runtime and npm to ensure successful installation.
    - `node -v`
    - `npm -v`

![node-version](/images/node-version.png)

## Go (Golang)
Used to develop the Chaincode

1. fetch the compressed Go language package specifically built for the Linux 64-bit (AMD64)
    - `wget https://golang.org/dl/go1.17.6.linux-amd64.tar.gz`

![install-go](/images/install-go.png)

2. Extracts the contents of the downloaded tarball
    - `sudo tar -xvf go1.17.6.linux-amd64.tar.gz`

![go-unzip](/images/go-unzip.png)

3. Moves the extracted go directory to the standard installation location
    - `sudo mv go /usr/local`

4. Permanently adds the Go binary directory (/usr/local/go/bin) to the system's $PATH environment variable.
    - `echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.profile`
    - `source ~/.profile`
5. Verify Installation
    - `go version`

![go-version](/images/go-version.png)

## Docker Compose
1. Install the docker-compose utility
    - `sudo apt install docker-compose -y`
2. Verify Installation
    - `docker-compose --version`

![docker-compose-version](/images/docker-compose-version.png)

## Weft
The **@hyperledgendary/weftility** package is a Node.js tool used to interact with the IBM Microfab deployment.

1. Installs the weft package globally
    - `sudo npm install -g @hyperledgendary/weftility`

![install-weft](/images/install-weft.png)

2. Verify Installation
    - `weft --version`

![weft-version](/images/weft-version.png)

# Step 2: Bootstrapping the Network using IBM Microfab
Microfab is a containerized Hyperledger Fabric runtime for use in development environments.

## On Terminal 1 Run the Network on Docker

### Deploy and Start Microfab
1. Create the project working directory and copy the configuration file
    - [Microfab Config for Charity Donations: config.json](/documents/config.json)
    - `cp /mnt/d/Hyperledger-Fabric-Charity-Donation/documents/config.json /home/neel/`
    - `mkdir HFN-Charity`
    - `cd HFN-Charity`
    - `cp ~/config.json ./config.json`

2. Run Microfab Container
    - The network is launched as a single Docker container, exposing all components (Peers, Orderer, CAs) via port 8080 on the host machine.
    - `export MICROFAB_CONFIG=$(cat config.json)`

![microfab-config-copy](/images/microfab-config-copy.png)

- Docker Run
    - `docker run -e MICROFAB_CONFIG -p 8080:8080 ibmcom/ibp-microfab`

> IBM Microfab: [https://asciinema.org/a/519913](https://asciinema.org/a/519913)

![microfab-docker-run](/images/microfab-docker-run.png)

## On Terminal 2 Run the Project to setup the environment variables and binaries

1. Extract Network Cryptography (Wallets, MSPs, Gateways)
    - This command downloads identities (certificates and keys) and connection profiles.
    - `curl -s http://console.127-0-0-1.nip.io:8080/ak/api/v1/components | \
weft microfab -w ./_wallets -p ./_gateways -m ./_msp -f`

![certificates-wallets-gateway-profiles](/images/certificates-wallets-gateway-profiles.png)
![environment-variables](/images/environment-variables.png)
[Environment Variables copied from the output](/documents/environment_variables.txt)

2. Execute them

![env-variables](/images/env-variables.png)

3. Fabric Tools Installation (This might not work)
    - `curl -sSL https://raw.githubusercontent.com/hyperledger/fabric/main/scripts/install-fabric.sh | bash -s -- binary`

![HLF-binary](/images/HLF-binary.png)

4. Environment Setup
    - `export PATH=$PATH:${PWD}/bin`
    - `export FABRIC_CFG_PATH=${PWD}/config`

![bin-env](/images/bin-env.png)

## Create the HFN-Charity-Contract Project to write smart contract

1. Chaincode Project Setup
    - `mkdir HFN-Charity-Contract`
    - `go mod init charity` (It initializes a Go module named charity, creating a go.mod file. This manages the dependencies for your chaincode)

2. Go source file (charity.go), which contains the chaincode logic (the Smart Contract implementation)
    - [Chaincode: charity.go](documents\charity.go)
    - `cp /mnt/d/Hyperledger-Fabric-Charity-Donation/documents/charity.go /home/neel/HFN-Charity/HFN-Charity-Contract`

3. Install the Hyperledger Fabric Contract API package
    - `go get github.com/hyperledger/fabric-contract-api-go@v1.2.1`

![external-package-install](/images/external-package-install.png)

4. To ensure the list of required dependencies is accurate and clean
    - `go mod tidy`

![go-mod-tidy](/images/go-mod-tidy.png)

5. Review the file
    - `cat go.mod`

![cat-go-mod](/images/cat-go-mod.png)

## On Terminal 2, Deploy the chaincode

1. Package the chaincode
    - `peer lifecycle chaincode package charity.tgz --path ./ --lang golang --label charity_1`

### Error
```
peer: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.34' not found (required by peer)
peer: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.32' not found (required by peer)
```

### Solution

1. Cleanup Incompatible Binaries:
    - `cd ..`
    - `rm -rf bin`

2. Download Stable v2.2.9 Binaries:
    - `wget https://github.com/hyperledger/fabric/releases/download/v2.2.9/hyperledger-fabric-linux-amd64-2.2.9.tar.gz`
    - `tar -xzf hyperledger-fabric-linux-amd64-2.2.9.tar.gz`
    - `wget https://github.com/hyperledger/fabric-ca/releases/download/v1.5.15/hyperledger-fabric-ca-linux-amd64-1.5.15.tar.gz`
    - `tar -xzf hyperledger-fabric-ca-linux-amd64-1.5.15.tar.gz`

3. Update PATH
    - `export PATH=$PATH:~/HFN-Charity/bin`

4. Check Version
    - `peer version`

![bin-path](/images/bin-path.png)

5. Re-package Chaincode (Success)
    - `peer lifecycle chaincode package charity.tgz --path ./HFN-Charity-Contract/ --lang golang --label charity_1`

![package-chaincode](/images/package-chaincode.png)

## Chaincode Lifecycle and Deployment (DonorOrg)

1. **Set Organization Context:** Before invoking or querying, make sure your environment points to the org/peer you want
    - `cd ~/HFN-Charity/HFN-Charity-Contract`
    - `export CORE_PEER_LOCALMSPID=DonorOrgMSP`
    - `export CORE_PEER_MSPCONFIGPATH=/home/neel/HFN-Charity/_msp/DonorOrg/donororgadmin/msp`
    - `export CORE_PEER_ADDRESS=donororgpeer-api.127-0-0-1.nip.io:8080`

2. Check Channel Membership: Confirms the Peer is joined to the target channel
    - `peer channel list`

![peer-channel-list](/images/peer-channel-list.png)

3. Install Package
    - `peer lifecycle chaincode install charity.tgz`

![install-chaincode](/images/install-chaincode.png)

> **Note:** This command uploads the package to the Peer and returns the Chaincode Package ID, which uniquely identifies the installed code.

4. Set Package ID
    - `export CC_PACKAGE_ID=charity_1:a838a19793bb05b124f84f39bbcb1fc8841977e3c698020a92122c8363eb9e7c`

5. **Approve Definition:** The DonorOrg formally approves the chaincode version, sequence, and package ID.
    - `peer lifecycle chaincode approveformyorg -o orderer-api.127-0-0-1.nip.io:8080 \
--channelID charity-channel --name charity --version 1 --sequence 1 --waitForEvent --package-id ${CC_PACKAGE_ID}`

![chaincode-approve](/images/chaincode-approve.png)

6. **Commit Definition:** Since Microfab often runs with a minimal endorsement policy for the initial commitment (typically requiring only one organization), the definition is committed to the channel.
    - `peer lifecycle chaincode commit -o orderer-api.127-0-0-1.nip.io:8080 --channelID charity-channel --name charity --version 1 --sequence 1`

![chaincode-commit](/images/chaincode-commit.png)


# Step 3: Transaction Execution and Testing

## Terminal 1 (DonorOrg)

### Invoke CreateDonation (add a new donation)

```
peer chaincode invoke -o orderer-api.127-0-0-1.nip.io:8080 --channelID charity-channel --name charity --waitForEvent -c '{"function":"CreateDonation","Args":["D001","Alice","HelpingHands","Education","2025-10-22","500"]}'
```

![create-donation](/images/create-donation.png)

### Query donation by ID (GetDonation)

```
peer chaincode query --channelID charity-channel --name charity -c '{"function":"GetDonation","Args":["D001"]}'
```

![get-donation](/images/get-donation.png)

### Update donation status (UpdateStatus)
```
peer chaincode invoke -o orderer-api.127-0-0-1.nip.io:8080 --channelID charity-channel --name charity --waitForEvent -c '{"function":"UpdateStatus","Args":["D001","Processed"]}'
```

![update-status](/images/update-status.png)

### Then query it again to confirm the status changed
```
peer chaincode query --channelID charity-channel --name charity -c '{"function":"GetDonation","Args":["D001"]}'
```

![status-changed](/images/status-changed.png)

### Query all donations (GetAllDonations)

```
peer chaincode query --channelID charity-channel --name charity -c '{"function":"GetAllDonations","Args":[]}'
```

![get-all-donations](/images/get-all-donations.png)

## Terminal 2 (NGOOrg)

- Change Directory
    - `cd HFN-Charity/`
    - `cd HFN-Charity-Contract/`

1. Set environment variables for NGOOrg
    - `export CORE_PEER_LOCALMSPID=NGOOrgMSP`
    - `export CORE_PEER_MSPCONFIGPATH=/home/neel/HFN-Charity/_msp/NGOOrg/ngoorgadmin/msp`
    - `export CORE_PEER_ADDRESS=ngoorgpeer-api.127-0-0-1.nip.io:8080`

2. Set PATH variables
    - `export PATH=$PATH:/home/neel/HFN-Charity/bin`
    - `export FABRIC_CFG_PATH=/home/neel/HFN-Charity/HFN-Charity-Contract/config`

3. Check channel membership
    - `peer channel list`

3. Install Package
    - `peer lifecycle chaincode install charity.tgz`
    `export CC_PACKAGE_ID=charity_1:a838a19793bb05b124f84f39bbcb1fc8841977e3c698020a92122c8363eb9e7c`

4. Approve Definition
```
`peer lifecycle chaincode approveformyorg -o orderer-api.127-0-0-1.nip.io:8080 \
--channelID charity-channel --name charity --version 1 --sequence 1 --waitForEvent --package-id ${CC_PACKAGE_ID}`
```

### Query donation by ID

```
peer chaincode query --channelID charity-channel --name charity -c '{"function":"GetDonation","Args":["D001"]}'
```

![query-donation-by-id](/images/query-donation-by-id.png)

### Update donation status (NGO approves/marks as processed)

```
peer chaincode invoke -o orderer-api.127-0-0-1.nip.io:8080 --channelID charity-channel --name charity --waitForEvent -c '{"function":"UpdateStatus","Args":["D001","ProcessedByNGO"]}'
```

![ngo-approve](/images/ngo-approve.png)

### Query all donations

```
peer chaincode query --channelID charity-channel --name charity -c '{"function":"GetAllDonations","Args":[]}'
```

![query-all-donations](/images/query-all-donations.png)

## Terminal 3 (AuditorOrg)

- Change Directory
    - `cd HFN-Charity/`
    - `cd HFN-Charity-Contract/`

1. Set environment variables for NGOOrg
    - `export CORE_PEER_LOCALMSPID=AuditorOrgMSP`
    - `export CORE_PEER_MSPCONFIGPATH=/home/neel/HFN-Charity/_msp/AuditorOrg/auditororgadmin/msp`
    - `export CORE_PEER_ADDRESS=auditororgpeer-api.127-0-0-1.nip.io:8080`

2. Set PATH variables
    -  `export PATH=$PATH:/home/neel/HFN-Charity/bin`
    - `export FABRIC_CFG_PATH=/home/neel/HFN-Charity/HFN-Charity-Contract/config`

3. Check channel membership
    - `peer channel list`

3. Install Package
    - `peer lifecycle chaincode install charity.tgz`
    - `export CC_PACKAGE_ID=charity_1:a838a19793bb05b124f84f39bbcb1fc8841977e3c698020a92122c8363eb9e7c`

4. Approve Definition
```
peer lifecycle chaincode approveformyorg -o orderer-api.127-0-0-1.nip.io:8080 \
--channelID charity-channel --name charity --version 1 --sequence 1 --waitForEvent --package-id ${CC_PACKAGE_ID}
```

## Finalizing the Donation Record by Auditor

```
peer chaincode invoke -o orderer-api.127-0-0-1.nip.io:8080 --channelID charity-channel --name charity --waitForEvent -c '{"function":"UpdateStatus","Args":["D001","Audited"]}'
```

![audit](/images/audit.png)

# Acknowledgements

* The implementation approach was developed after seeing the **demo lecture conducted by Professor Mrs. Lifna** [https://github.com/LifnaJos/Hyperledger-Fabric-Network-Go-Lang](https://github.com/LifnaJos/Hyperledger-Fabric-Network-Go-Lang), as part of Blockchain course under the Computer Engineering Department, V.E.S. Institute of Technology, Mumbai. 

* [https://github.com/hyperledger-labs/microfab](https://github.com/hyperledger-labs/microfab)

* [https://hyperledger-fabric.readthedocs.io/en/release-2.5/](https://hyperledger-fabric.readthedocs.io/en/release-2.5/)