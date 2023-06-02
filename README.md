# How-to-implement-test-and-deploy-NFT-Smart-Contract-with-Hardhat-on-celo-Blockchain
## Table of Contents

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [What You Will Learn After This Tutorial](#what-you-will-learn-after-this-tutorial)
- [Step 1: Set up the development environment](#step-1-set-up-the-development-environment)
- [Step 2: Implement the NFT smart contract](#step-2-implement-the-nft-smart-contract)
  - [1. Implementing the mint function](#1-implementing-the-mint-function)
  - [2. Full Smart Contract implementation](#2-full-smart-contract-implementation)
- [Step 3: Configure Hardhat](step-3-onfigure-hardhat)
- [Step 4: Write tests for the NFT smart contract](#step-4-write-tests-for-the-nft-smart-contract)
- [Step 5: Deploy the NFT smart contract on Celo](#step-5-deploy-the-nft-smart-contract-on-celo)
- [Conclusion](#conclusion)

## Introduction
The emergence of blockchain technology has brought attention to various terms such as Smart Contracts and NFTs. However, it is important to understand the actual meaning behind these terms.

Smart contracts refer to programs stored on a blockchain that execute when specific predetermined conditions are met. These contracts can store small amounts of data using common data structures, which is crucial for tokenization use cases. Tokenization involves mapping token identifiers to owner identifiers, enabling tracking of ownership for different tokens.

NFTs, or non-fungible tokens, are tokens on the blockchain that represent unique assets like artwork, digital content, or media. Each NFT serves as an immutable digital certificate of ownership and authenticity for a specific asset, regardless of whether it is digital or physical.

In this tutorial, we will guide you through the process of implementing, testing, and deploying NFT smart contracts. 
To accomplish this, we will utilize the following tools:

- `Hardhat`: a developer tool for scaffolding smart contract projects and providing a development environment for Ethereum-based applications. Hardhat is known for its extensibility, built-in testing capabilities, and support for task automation.

- `OpenZeppelin`: a widely used smart contract library that offers a range of utility functions and pre-built contract templates. It provides secure and audited implementations of various standards, such as ERC20 and ERC721, making it easier to develop robust and standards-compliant smart contracts.

- `Alfajores`: an EVM-compatible blockchain developed by Celo. It serves as a testnet for Celo, allowing developers to experiment and deploy their smart contracts in a controlled environment before deploying to the main Celo network. Alfajores offers a low-cost and developer-friendly platform for testing and showcasing applications built on Celo.

## Prerequisites:

- Basic understanding of Solidity and smart contracts.
- Node.js and npm (Node Package Manager) installed on your machine.
- Familiarity with the command line interface (CLI).

## What You Will Learn After This Tutorial

- how to write a smart contract that can be used to create NFTs
- write tests for our smart contract
- deployed it to the alfajores testnet

## Step 1: Set up the development environment

1. Install `Hardha`t: Open your terminal and run the following command to install Hardhat globally:
  ```bash
  npm install -g hardhat
  ```
2. Create a new directory for your project and navigate to it:
 ```bash
  mkdir nft-contract && cd nft-contract 
 ```
3. Initialize `Hardhat`: Run the following command to initialize a new Hardhat project:
 ```bash
  npx hardhat
```
Follow the on-screen prompts and choose the "Create a basic sample project" option.
![Image Description](https://i.ibb.co/fdrB19R/Screenshot-from-2023-06-01-17-49-54.png)
4. Install necessary dependencies: Install the required npm packages by running the following command:
 ```bash
  npm install @openzeppelin/contracts hardhat-ethers ethers dotenv
```

## Step 2: Implement the NFT smart contract
Let’s create a new file named NFT.sol inside the contracts directory. This is going to be our first smart contract which will help us in minting and updating NFTs.
 
 ```Solidity

// SPDX-License-Identifier: MIT
pragma solidity  ^0.8.0

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract NFT is ERC721, Ownable {
    address public _contractOwner;
    uint256 public tokenCounter;
    
    mapping(uint => string) public tokenURIMap;
    mapping(uint => uint) public priceMap;

    event Minted(address indexed minter, uint price, uint nftID, string tokenURI);
        
    event PriceUpdate(address indexed owner, uint oldPrice, uint newPrice, uint nftID);

    constructor() ERC721("MyNFT", "MNFT") {
        _contractOwner = msg.sender;
        tokenCounter = 1;
    }
 ```
 As you can see, here we are doing the following:

- Importing OpenZeppelin ERC721 standard and All the ownable utility functions
Inheriting the OpenZeppelin ERC721 smart contract into our `NFT.sol` smart contract using the `is` keyword.
- Defining variables for holding the contract owner's address and the one for keeping track of the NFT count.
- Defining mapping for NFT price & token URI, given they are information which needs to be persisted on the blockchain.
- Defining the `Minted & PriceUpadte` events that will be emitted when the `mint & priceUpadte` will be fired.
- Given the constructor is always the first function that is called while deploying a smart contract. We are providing directly the NFT name as `MyNFT` and `MNFT` as the symbol. The `msg.sender` is a keyword which returns the address of the account invoking the smart contract and we are setting the initial token count as one.

 ### 1. Implementing the mint function
```Solidity
function mint(string memory _uri, address _toAddress, uint _price) public returns (uint){
  uint _tokenId = tokenCounter;
  priceMap[_tokenId] = _price;

  _safeMint(_toAddress, _tokenId);
  _setTokenURI(_tokenId, _uri);

  tokenCounter++;

  emit Minted(_toAddress, _price, _tokenId, _uri);

  return _tokenId;
}

function _setTokenURI(uint256 _tokenId, string memory _uri) internal virtual {
   require(_exists(_tokenId),"ERC721Metadata: URI set of nonexistent token"); 
   tokenURIMap[_tokenId] = _uri;
}
```
In the above mint function, we are persisting only crucial NFT attributes on blockchain in order to minimize the cost since storing data directly on a blockchain has an associated cost, it won’t be financially feasible if all the NFT metadata is stored on blockchain, the rest of NFT metadata can be stored on a centralised database pointing to the NFT data stored on blockchain using token URI attribute. After setting the token id and price, we are calling the `_safeMint` method from OpenZeppelin ERC721 that literally mints the new NFT. The method takes two parameters, the minter address and the token id of the new-minted NFT.

The `_setTokenURI` is acting as a helper method for setting the token URI, by adding to it the `internal` visibility modifier will make it available only inside this contract and its derived contracts. For the record, this method used to be part of the OpenZeppelin ERC721 utility methods. In the end, we are incrementing the token id, emitting the event and returning the token id.

## 2. Full Smart Contract implementation

```Solidity
// SPDX-License-Identifier: MIT
pragma solidity  ^0.8.0

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract NFT is ERC721, Ownable {
    address public _contractOwner;
    uint256 public tokenCounter;
    
    mapping(uint => string) public tokenURIMap;
    mapping(uint => uint) public priceMap;

    event Minted(address indexed minter, uint price, uint nftID, string uri);
        
    event PriceUpdate(address indexed owner, uint oldPrice, uint newPrice, uint nftID);

    constructor() ERC721("MyNFT", "MNFT") {
        _contractOwner = msg.sender;
        tokenCounter = 1;
    }

    function mint(string memory _uri, address _toAddress, uint _price) public returns (uint){
        uint _tokenId = tokenCounter;
        priceMap[_tokenId] = _price;

        _safeMint(_toAddress, _tokenId);
        _setTokenURI(_tokenId, _uri);

        tokenCounter++;

        emit Minted(_toAddress, _price, _tokenId, _uri);

        return _tokenId;
    }

    function _setTokenURI(uint256 _tokenId, string memory _tokenURI) internal virtual {
        require(_exists(_tokenId),"ERC721Metadata: URI set of nonexistent token"); 
        tokenURIMap[_tokenId] = _tokenURI;
    }

}
```

## Step 3: Configure Hardhat

1. Open the `hardhat.config.js` file in the project root directory and replace its contents with the following code:

```javascript
require("@nomiclabs/hardhat-waffle");
require("dotenv").config({ path: ".env" });

module.exports = {
  solidity: "0.8.4",
  networks: {
    alfajores: {
      url: "https://alfajores-forno.celo-testnet.org", 
      accounts: [process.env.PRIVATE_KEY],
      chainId: 44787,
    },
  },
  mocha: {
    timeout: 500000, // 500 seconds max for running tests
}
};
```
This configuration file sets up the networks to be used with Hardhat, including the Celo network. It retrieves the necessary configuration values from the `.env file`, which we will set up in the next step.

2. Create a new file named .env in the project root directory and add the following lines, replacing the placeholder values with your own:

```makefile
PRIVATE_KEY=<Your_private_key>
```
Replace `<Your_private_key>` with your private key.

## Step 4: Write tests for the NFT smart contract

1. Open the `test` folder in the project directory and create a new file named `MyNFT.test.js`.
 Add the following code:
 ```javascript
const { expect } = require('chai');
const {ethers} = require("hardhat");

describe('NFT', function () {
  let nftContract;
  let owner;
  let addr1;

  beforeEach(async function () {
    const NFT = await ethers.getContractFactory('NFT');
    owner = (await ethers.getSigners())[0]; // Retrieve owner as a signer
    addr1 = '0x60031b5df905D92786dea1781E731B88b959c8A6'

    nftContract = await NFT.deploy();
    await nftContract.deployed();
  });

  it('Should mint a new NFT', async function () {
    const recipient = addr1;
    const tokenURI = 'https://example.com/token-metadata.json';
    const price = ethers.utils.parseEther('0.1');

    await nftContract.connect(owner).mint(tokenURI, recipient, price);
    const tokenId = await nftContract.tokenCounter();

    expect(await nftContract.ownerOf(tokenId)).to.equal(recipient);
    expect(await nftContract.tokenURIMap(tokenId)).to.equal(tokenURI);
    expect(await nftContract.priceMap(tokenId)).to.equal(price);
  });
});

 ```
 
This code sets up a basic test suite for the MyNFT contract.
It verifies that a new NFT can be minted and checks the owner and token URI.

## Step 5: Deploy the NFT smart contract on Celo
Open the `scripts` folder in the project directory and replace the code in the `deploy.js` file by the code below
```javascript
const hre = require("hardhat");

async function main() {
  const nftContract = await hre.ethers.getContractFactory("NFT");
  const nft = await nftContract.deploy();
  await nft.deployed();
  console.log(
    `NFT contract deployed to ${nft.address}`
  );
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});

```

1. In your terminal, run the following command to compile the contract:
```bash 
npx hardhat compile
```
2. Deploy the contract to the Celo network by running the following command:
```bash
npx hardhat run --network alfajores scripts/deploy.js
```
This command executes the deployment script, which is automatically created by Hardhat in the `scripts` folder.
3. Test the smart contract by running the command below
```bash
npx hardhat test --network alfajores
```
You should be able to get the following result:
![test terminal result](https://i.ibb.co/47prSMZ/Screenshot-from-2023-06-02-02-19-51.png)

**Congratulations! You have successfully implemented, tested, and deployed an NFT smart contract with Hardhat on the Celo blockchain**. You can now interact with the contract using the generated artifacts and the deployed contract address.

## Conclusion
In this tutorial, we learned how write a smart contract that can be used to create NFTs, we did also write tests for our smart contract and finally deployed it to the alfajores testnet. Given Alfajores`: an EVM-compatible blockchain developed by Celo, the same procedures can be used to build and deploy a DAPP to any EVM compatible network.

The link to my project repository can be found [nft-tutorial](https://github.com/Muhindo-Galien/nft-tutorial)

