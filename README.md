# How-to-implement-test-and-deploy-NFT-Smart-Contract-with-Hardhat-on-celo-Blockchain
## Table of Contents

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [What You Will Learn After This Tutorial](#what-you-will-learn-after-this-tutorial)
- [Step 1: Set up the development environment](#step-1-set-up-the-development-environment)
- [Step 2: Implement the NFT smart contract](#step-2-implement-the-nft-smart-contract)
  - [1. Functions Breakdown and Explanation](#1-functions-breakdown-and-explanation)
     - [mint Function](#mint-function)
     - [_setTokenURI Function](#_settokenuri-function)
     - [getTokenURI Function](#gettokenuri-function)
     - [getTokenPrice Function](#gettokenprice-function)
     - [setTokenPrice Function](#settokenprice-function)
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

1. Install `Hardhat`: Open your terminal and run the following command to install Hardhat globally:
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
pragma solidity  ^0.8.17;

/**
 @ dev - visit https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/ERC721.sol to find the full code
**/
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
/**
 @ dev - visit https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable.sol to find the full code
**/
import "@openzeppelin/contracts/access/Ownable.sol";


/**
 * @title NFT contract
 * @notice This contract provides functionality for minting and managing non-fungible tokens (NFTs).
 */
contract NFT is ERC721, Ownable {

    /**
     @dev Counter for tracking the number of tokens minted 
    */
    uint256 private tokenCounter;

    /// tokenURIMap - Mapping of token IDs to their corresponding URI strings    
    mapping(uint => string) public tokenURIMap;

    /// priceMap - Mapping of token IDs to their corresponding prices
    mapping(uint => uint) public priceMap;


    /** @dev Event triggered when a token is minted
    *  @param minter The address of the minter who minted the token
    *  @param price The price of the token
    *  @param nftID The ID of the minted token
    *  @param uri The URI of the token's metadata
    */
    event Minted(address indexed minter, uint price, uint nftID, string uri);

     /** @dev Event triggered when the price of a token is updated
     *  @param owner The address of the token owner
     *  @param oldPrice The old price of the token
     *  @param newPrice The new price of the token
     *  @param nftID The ID of the token
     */   
    event PriceUpdate(address indexed owner, uint oldPrice, uint newPrice, uint nftID);

     /** @dev Contract constructor
     *  It initializes the NFT contract with the specified name and symbol, and sets the initial value of tokenCounter to 1
     */
    constructor() ERC721("MyNFT", "MNFT") {

        tokenCounter = 1;

    }
 }
 ```
 As you can see, here we are doing the following:

- Importing OpenZeppelin ERC721 standard and All the ownable utility functions
Inheriting the OpenZeppelin ERC721 smart contract into our `NFT.sol` smart contract using the `is` keyword.
- Defining variables for holding the contract owner's address and the one for keeping track of the NFT count.
- Defining mapping for NFT price & token URI, given they are information which needs to be persisted on the blockchain.
- Defining the `Minted & PriceUpadte` events that will be emitted when the `mint & priceUpadte` will be fired.
- Given the constructor is always the first function that is called while deploying a smart contract. We are providing directly the NFT name as `MyNFT` and `MNFT` as the symbol. The `msg.sender` is a keyword which returns the address of the account invoking the smart contract and we are setting the initial token count as one.

 ### 1. Functions Breakdown and Explanation
```Solidity
     /** @dev Mint a new token and assign it to the specified address
     *  @param _uri The URI of the token's metadata
     *  @param _toAddress The address to which the minted token will be assigned
     *  @param _price The price of the minted token
     *  @return mintedTokenID The ID of the minted token
     */
    function mint(string memory _uri, address _toAddress, uint _price) public onlyOwner returns (uint mintedTokenID){

        uint _tokenId = tokenCounter;

        priceMap[_tokenId] = _price;

        _safeMint(_toAddress, _tokenId);

        _setTokenURI(_tokenId, _uri);

        tokenCounter++;

        emit Minted(_toAddress, _price, _tokenId, _uri);

        return _tokenId;
    }

     /** 
     * @dev Set the URI of the specified token
     *  @param _tokenId The ID of the token
     *  @param _tokenURI The URI of the token's metadata
     */
    function _setTokenURI(uint256 _tokenId, string memory _tokenURI) internal virtual {
        require(_exists(_tokenId),"ERC721Metadata: URI set of nonexistent token"); 
        tokenURIMap[_tokenId] = _tokenURI;
    }

     /** 
      * @dev Get the URI of the specified token
      *  @param _tokenId The ID of the token
      *  @return The URI of the token's metadata
      */
    function getTokenURI(uint256 _tokenId) public view returns (string memory) {
        require(_exists(_tokenId), "ERC721Metadata: URI query for nonexistent token");
        return tokenURIMap[_tokenId];
    }

     /**
     * @dev Get the price of the specified token
     *  @param _tokenId The ID of the token
     *  @return priceMapTokenID The price of the token
     */
    function getTokenPrice(uint256 _tokenId) public view returns (uint256 priceMapTokenID) {
        require(_exists(_tokenId), "ERC721Metadata: Price query for nonexistent token");
        return priceMap[_tokenId];
    }

     /** @dev Set the price of the specified token
     *  @param _tokenId The ID of the token
     *  @param _newPrice The new price for the token
     */
    function setTokenPrice(uint256 _tokenId, uint256 _newPrice) public onlyOwner {
        require(_exists(_tokenId), "ERC721Metadata: Price set for nonexistent token");
        uint256 _oldPrice = priceMap[_tokenId];
        priceMap[_tokenId] = _newPrice;

        emit PriceUpdate(owner(), _oldPrice, _newPrice, _tokenId);
    }

```

### `mint` Function
In the above mint function, we are persisting only crucial NFT attributes on blockchain in order to minimize the cost since storing data directly on a blockchain has an associated cost, it won’t be financially feasible if all the NFT metadata is stored on blockchain, the rest of NFT metadata can be stored on a centralised database pointing to the NFT data stored on blockchain using token URI attribute. After setting the token id and price, we are calling the `_safeMint` method from OpenZeppelin ERC721 that literally mints the new NFT. The method takes two parameters, the minter address and the token id of the new-minted NFT.

### `_setTokenURI` Function

The `_setTokenURI` is acting as a helper method for setting the token URI, by adding to it the `internal` visibility modifier will make it available only inside this contract and its derived contracts. For the record, this method used to be part of the OpenZeppelin ERC721 utility methods. In the end, we are incrementing the token id, emitting the event and returning the token id.

### `getTokenURI` function
The purpose of the `getTokenURI` function is to retrieve the URI (Uniform Resource Identifier) of a specific token. In ERC721 tokens, the URI is typically used to store metadata associated with the token, such as its name, image, description, or other relevant information.

Here's how the function works:
- It takes `_tokenId` as a parameter, representing the ID of the token for which the URI is being queried.
- The function checks if the token with the specified `_tokenId` exists by calling the `_exists` function. If the token doesn't exist, it reverts the transaction with an error message.
- If the token exists, the function retrieves the URI associated with the `_tokenId` from the `tokenURIMap` mapping.
- Finally, it returns the URI as a `string` to the caller.

This function is useful when you want to retrieve the metadata associated with a specific token, such as accessing an external resource (e.g., an image or description) based on the URI provided.

### `getTokenPrice` Function

The `getTokenPrice` function is used to retrieve the price of a specific token. It allows you to get the current price associated with a token ID stored in the `priceMap` mapping.

Here's how the function works:
- It takes `_tokenId` as a parameter, representing the ID of the token for which the price is being queried.
- The function checks if the token with the specified `_tokenId` exists by calling the `_exists` function. If the token doesn't exist, it reverts the transaction with an error message.
- If the token exists, the function retrieves the price associated with the `_tokenId` from the `priceMap` mapping.
- Finally, it returns the price as a `uint256` to the caller.

This function is useful when you want to retrieve the price of a specific token, such as for displaying the price to users or performing price-related operations within your smart contract's logic.

### `setTokenPrice` Function
The setTokenPrice function allows the contract owner to set the price for a specific token. It is used to update the price associated with a token ID in the priceMap mapping.

Here's how the function works:

It takes _tokenId and _newPrice as parameters, representing the ID of the token and the new price to be set, respectively.
The function checks if the token with the specified _tokenId exists by calling the _exists function. If the token doesn't exist, it reverts the transaction with an error message.
If the token exists, the function first retrieves the old price associated with the _tokenId from the priceMap mapping and stores it in the _oldPrice variable.
Then, it updates the priceMap mapping with the new price _newPrice for the specified _tokenId.
Finally, it emits a PriceUpdate event to notify listeners about the price change, including the contract owner, the old price, the new price, and the token ID.
This function is designed to be called only by the contract owner, ensuring that only the owner has the authority to update the token prices. It provides a way to manage and update the prices associated with specific tokens within the contract.







## 2. Full Smart Contract implementation

```Solidity
// SPDX-License-Identifier: MIT
pragma solidity  ^0.8.17;

/**
 @ dev - visit https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/ERC721.sol to find the full code
**/
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
/**
 @ dev - visit https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable.sol to find the full code
**/
import "@openzeppelin/contracts/access/Ownable.sol";


/**
 * @title NFT contract
 * @notice This contract provides functionality for minting and managing non-fungible tokens (NFTs).
 */
contract NFT is ERC721, Ownable {

    /**
     @dev Counter for tracking the number of tokens minted 
    */
    uint256 private tokenCounter;

    /// tokenURIMap - Mapping of token IDs to their corresponding URI strings    
    mapping(uint => string) public tokenURIMap;

    /// priceMap - Mapping of token IDs to their corresponding prices
    mapping(uint => uint) public priceMap;


    /** @dev Event triggered when a token is minted
    *  @param minter The address of the minter who minted the token
    *  @param price The price of the token
    *  @param nftID The ID of the minted token
    *  @param uri The URI of the token's metadata
    */
    event Minted(address indexed minter, uint price, uint nftID, string uri);

     /** @dev Event triggered when the price of a token is updated
     *  @param owner The address of the token owner
     *  @param oldPrice The old price of the token
     *  @param newPrice The new price of the token
     *  @param nftID The ID of the token
     */   
    event PriceUpdate(address indexed owner, uint oldPrice, uint newPrice, uint nftID);

     /** @dev Contract constructor
     *  It initializes the NFT contract with the specified name and symbol, and sets the initial value of tokenCounter to 1
     */
    constructor() ERC721("MyNFT", "MNFT") {

        tokenCounter = 1;

    }

     /** @dev Mint a new token and assign it to the specified address
     *  @param _uri The URI of the token's metadata
     *  @param _toAddress The address to which the minted token will be assigned
     *  @param _price The price of the minted token
     *  @return mintedTokenID The ID of the minted token
     */
    function mint(string memory _uri, address _toAddress, uint _price) public onlyOwner returns (uint mintedTokenID){

        uint _tokenId = tokenCounter;

        priceMap[_tokenId] = _price;

        _safeMint(_toAddress, _tokenId);

        _setTokenURI(_tokenId, _uri);

        tokenCounter++;

        emit Minted(_toAddress, _price, _tokenId, _uri);

        return _tokenId;
    }

     /** 
     * @dev Set the URI of the specified token
     *  @param _tokenId The ID of the token
     *  @param _tokenURI The URI of the token's metadata
     */
    function _setTokenURI(uint256 _tokenId, string memory _tokenURI) internal virtual {
        require(_exists(_tokenId),"ERC721Metadata: URI set of nonexistent token"); 
        tokenURIMap[_tokenId] = _tokenURI;
    }

     /** 
      * @dev Get the URI of the specified token
      *  @param _tokenId The ID of the token
      *  @return The URI of the token's metadata
      */
    function getTokenURI(uint256 _tokenId) public view returns (string memory) {
        require(_exists(_tokenId), "ERC721Metadata: URI query for nonexistent token");
        return tokenURIMap[_tokenId];
    }

     /**
     * @dev Get the price of the specified token
     *  @param _tokenId The ID of the token
     *  @return priceMapTokenID The price of the token
     */
    function getTokenPrice(uint256 _tokenId) public view returns (uint256 priceMapTokenID) {
        require(_exists(_tokenId), "ERC721Metadata: Price query for nonexistent token");
        return priceMap[_tokenId];
    }

     /** @dev Set the price of the specified token
     *  @param _tokenId The ID of the token
     *  @param _newPrice The new price for the token
     */
    function setTokenPrice(uint256 _tokenId, uint256 _newPrice) public onlyOwner {
        require(_exists(_tokenId), "ERC721Metadata: Price set for nonexistent token");
        uint256 _oldPrice = priceMap[_tokenId];
        priceMap[_tokenId] = _newPrice;

        emit PriceUpdate(owner(), _oldPrice, _newPrice, _tokenId);
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

You can learn more about openzeppelin smart contract from this url [Openzeppelin Official Website](https://www.openzeppelin.com/)

The link to my project repository can be found [nft-tutorial](https://github.com/Muhindo-Galien/nft-tutorial)

