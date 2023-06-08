# How to Implement, Test & Deploy NFT Smart Contract using Hardhat on Celo Blockchain:

## Table of Contents:

- [Introduction](#introduction)
- [Pre-requisites](#pre-requisites)
- [Learning Outcomes](#learning-outcomes)
- [Step 1: Set up the Development Environment](#step-1-set-up-the-development-environment)
- [Step 2: Implement the NFT Smart Contract](#step-2-implement-the-nft-smart-contract)
  - [1. Implementing the mint function](#1-implementing-the-mint-function)
  - [2. Full Smart Contract Implementation](#2-full-smart-contract-implementation)
- [Step 3: Configure Hardhat](step-3-onfigure-hardhat)
- [Step 4: Write tests for the NFT Smart Contract](#step-4-write-tests-for-the-nft-smart-contract)
- [Step 5: Deploy the NFT Smart Contract on Celo](#step-5-deploy-the-nft-smart-contract-on-celo)
- [Conclusion](#conclusion)

## Introduction:

The emergence of blockchain technology has brought attention to various terms such as Smart Contracts and NFTs etc. However, it is important to understand the actual meaning behind these terms.

Smart contracts refer to programs stored on a blockchain that executes when specific pre-determined conditions are met. These contracts can store small amounts of data using common data structures, which is crucial for tokenization use cases. Tokenization involves mapping token identifiers to owner identifiers, enabling tracking of ownership for different tokens.

NFTs, or non-fungible tokens, are tokens on the blockchain that represent unique assets like artwork, digital content or media. Each NFT serves as an immutable digital certificate of ownership and authenticity for a specific asset, regardless of whether it is digital or physical.

In this tutorial, we will guide you through the process of implementing, testing and deploying NFT Smart Contracts. 
To accomplish this, we will utilize the following tools:

- `Hardhat`: is a developer tool for scaffolding Smart Contract projects and providing a development environment for Ethereum-based applications. Hardhat is known for its extensibility, built-in testing capabilities and support for task automation.

- `OpenZeppelin`: is a widely used Smart Contract library that offers a range of utility functions and pre-built contract templates. It provides secure and audited implementations of various standards like ERC20 and ERC721, making it easier to develop robust and standards-compliant Smart Contracts.

- `Alfajores`: an EVM-compatible blockchain developed by Celo. It serves as a testnet for Celo, allowing developers to experiment and deploy their Smart Contracts in a controlled environment before deploying to the main Celo network. Alfajores offers a low-cost and developer-friendly platform for testing and showcasing applications built on Celo.

## Pre-requisites:

- Understanding of [Solidity](https://docs.soliditylang.org/en/v0.8.20/) and [Smart Contracts](https://www.ibm.com/topics/smart-contracts).
- Install [Node.js](https://nodejs.org/en) and [npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm) (Node Package Manager) 
- Familiarity with the Command Line Interface (CLI).

## Learning Outcomes:

- How to write a Smart Contract that can be used to create NFTs?
- Write tests for our Smart Contract.
- Deployed it to the alfajores testnet.

## Step 1: Set up the Development Environment:

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

## Step 2: Implement the NFT Smart Contract:
Let’s create a new file named "NFT.sol" inside the contracts directory. This is going to be our first Smart Contract which will help us in minting and updating NFTs.
 
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

 ### 1. Implementing the mint function:
 
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

## 2. Full Smart Contract Implementation:

```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract NFT is ERC721, Ownable {
    address public _contractOwner;
    uint256 public tokenCounter;

    // Maps token ID to its corresponding token URI
    mapping(uint => string) public tokenURIMap;

    // Maps token ID to its corresponding price
    mapping(uint => uint) public priceMap;

    // Event emitted when a new NFT is minted
    event Minted(address indexed minter, uint price, uint nftID, string uri);

    // Event emitted when the price of an NFT is updated
    event PriceUpdate(address indexed owner, uint oldPrice, uint newPrice, uint nftID);

    constructor() ERC721("MyNFT", "MNFT") {
        _contractOwner = msg.sender;
        tokenCounter = 1;
    }

    /**
     * @dev Mints a new NFT with the provided URI, assigns it to the given address,
     * and sets its price.
     * @param _uri The URI of the token's metadata
     * @param _toAddress The address to assign the minted NFT to
     * @param _price The price of the NFT
     * @return The ID of the minted NFT
     */
    function mint(string memory _uri, address _toAddress, uint _price) public returns (uint) {
        uint _tokenId = tokenCounter;
        priceMap[_tokenId] = _price;

        _safeMint(_toAddress, _tokenId);
        _setTokenURI(_tokenId, _uri);

        tokenCounter++;

        emit Minted(_toAddress, _price, _tokenId, _uri);

        return _tokenId;
    }

    /**
     * @dev Sets the token URI of the given token ID.
     * Only callable from within the contract.
     * @param _tokenId The ID of the token to set the URI for
     * @param _tokenURI The URI to set for the token
     */
    function _setTokenURI(uint256 _tokenId, string memory _tokenURI) internal virtual {
        require(_exists(_tokenId), "ERC721Metadata: URI set of nonexistent token");
        tokenURIMap[_tokenId] = _tokenURI;
    }

    /**
     * @dev Safely mints a new NFT to the given address.
     * This function performs error checking to prevent minting to the zero address
     * and prevents minting of an already existing token.
     * @param _to The address to assign the minted NFT to
     * @param _tokenId The ID of the token to be minted
     */
    function _safeMint(address _to, uint256 _tokenId) internal {
        require(_to != address(0), "ERC721: mint to the zero address");
        require(!_exists(_tokenId), "ERC721: token already minted");

        _mint(_to, _tokenId);
    }

    /**
     * @dev Checks whether a token with the given ID exists.
     * @param _tokenId The ID of the token to check
     * @return A boolean indicating whether the token exists or not
     */
    function _exists(uint256 _tokenId) internal view returns (bool) {
        return _exists(_tokenId);
    }

    /**
     * @dev Sets the token URI for the given token ID.
     * This function allows the token URI to be updated.
     * @param _tokenId The ID of the token to set the URI for
     * @param _tokenURI The URI to set for the token
     */
    function setTokenURI(uint256 _tokenId, string memory _tokenURI) public virtual {
        require(_exists(_tokenId), "ERC721Metadata: URI set of nonexistent token");
        tokenURIMap[_tokenId] = _tokenURI;
    }

    /**
     * @dev Updates the price of the given token ID.
     * Only callable by the contract owner.
     * @param _tokenId The ID of the token to update the price for
     * @param _newPrice The new price to set for the token
     */
    function updatePrice(uint256 _tokenId, uint _newPrice) public onlyOwner {
        require(_exists(_tokenId), "ERC721: Price update of nonexistent token");
        uint _oldPrice = priceMap[_tokenId];
        priceMap[_tokenId] = _newPrice;
        emit PriceUpdate(msg.sender, _oldPrice, _newPrice, _tokenId);
    }
}

```

## Step 3: Configure Hardhat:

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

## Step 4: Write tests for the NFT smart contract:

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

## Step 5: Deploy the NFT smart contract on Celo:

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

**Congratulations! You have successfully implemented, tested and deployed an NFT Smart Contract with Hardhat on the Celo blockchain**. You can now interact with the contract using the generated artifacts and the deployed contract address.

## Conclusion:

In this tutorial, we learned how write a Smart Contract that can be used to create NFTs, we did also write tests for our Smart Contract and finally deployed it to the alfajores testnet. Given Alfajores`: an EVM-compatible blockchain developed by Celo, the same procedures can be used to build and deploy a Dapp to any EVM compatible network.

The link to my project repository can be found [nft-tutorial](https://github.com/Muhindo-Galien/nft-tutorial)
