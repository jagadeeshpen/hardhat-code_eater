mocha --- Mocha is a JavaScript test framework running on Node.js .
hardhat -- its an develpoment environment for eth.
chai -- Chai is an assertion library that works seamlessly with Mocha (or other test frameworks) to provide expressive and flexible assertion.
- By using hardhat we can compile, test, debug and deploy.
- when are testing we use mocha javascript library.
- What is Hardhat -- Hardhat is a development environment to compile, deploy, test and debug your ethereum softwares.
- Hardhat is written in javascript.
# Installation of hardhat -- 
    1. npm init
    2. npm install --save-dev hardhat
    3. npx hardhat (select -- Create an empty hardhat.config.js)
    4. npm install --save-dev @nomicfoundation/hardhat-toolbox (for testing purpose we use this package)
    5. Add `require("@nomicfoundation/hardhat-toolbox");` to hardhat.config.js
- Then create contract folder for .sol files.
- Then create test folder for test.js files.
- Then create scripts folder for deploy.js files.
# Smart Contract.
- After creating folders write a contract on contracts folder.
### code 
```
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

// import "hardhat/console.sol";

contract Token {
    string public name = "HardHat Token";
    string public symbol = "HHT";
    uint256 public totalSupply = 10000;

    address public owner;

    mapping(address => uint256) balances;

    constructor() {
        balances[msg.sender] = totalSupply;
        owner = msg.sender;
    }

    function transfer(address to, uint256 amount) external {
        <!-- console.log("**Sender balance is %s tokens**", balances[msg.sender]);
        console.log(
            "**Sender is sending %s tokens to %s address**",
            amount,
            to
        ); -->

        require(balances[msg.sender] >= amount, "Not enough tokens");
        balances[msg.sender] -= amount; //balances[msg.sender]=balances[msg.sender]-amount;
        balances[to] += amount;
    }

    function balanceOf(address account) external view returns (uint256) {
        return balances[account];
    }
}
```
# Compiling
- Go to contracts folder and compile the contract. For compiling use `npx hardhat compile` 
- After compilation sucessfull two folders are created automatically..
  1. artifacts
  2. cache
- In artifacts folder we can find the ABI of our compile file under contracts folder.
# Testing
- To test the written smart contract we have to create an token.js file under the teset folder.
```
For testing smart contract we use chai library + mocha framework. (we write unit testing only in TDD approch)
```
```
Mocha and Chai are widely used and effective tools for testing smart contracts. Mocha provides a framework for organizing and executing tests, while Chai offers a variety of assertion styles for making test assertions more readable and expressive. Together, Mocha and Chai provide a powerful testing solution for verifying the behavior and correctness of smart contracts.
```
basic testing code
```
const{expect} = require("chai"); // for importing chai

// describe is from macha. by this we are creating a framework. function will increase the visibilty for testing.
// expect is from chai library.

describe("Token contract", function(){
    it("Deployment should assign the total supply of tokens to the owner", async function(){
        const [owner] = await ethers.getSigners();

        const Token = await ethers.getContractFactory("Token"); //instance contract

        const hardhatToken = await Token.deploy();//deploy contract        

        const ownerBalance = await hardhatToken.balanceOf(owner.address);//ownerBalance = 10000
       
        expect(await hardhatToken.totalSupply()).to.equal(ownerBalance);//totalSupply = 10000
    })

    it("Should transfer tokens between accounts", async function(){
        const [owner, addr1, addr2] = await ethers.getSigners();

        const Token = await ethers.getContractFactory("Token"); //instance contract

        const hardhatToken = await Token.deploy();//deploy contract

        //Transder 100 tokens from owner to add1
        await hardhatToken.transfer(addr1.address, 10);
        expect(await hardhatToken.balanceOf(addr1.address)).to.equal(10);

        //transfer 5 Tokens from addr1 to addr2
        await hardhatToken.connect(addr1).transfer(addr2.address, 5);
        expect(await hardhatToken.balanceOf(addr2.address)).to.equal(5);
    })
})
```

code for testing using mocha framework hooks ex.. beforeach
```
const { expect } = require("chai");

describe("Token Contract", function () {

  let Token;
  let hardhatToken;
  let owner;
  let addr1;
  let addr2;
  let addrs;

  beforeEach(async function () {
    Token = await ethers.getContractFactory("Token");
    [owner, addr1, addr2, ...addrs] = await ethers.getSigners();
    hardhatToken = await Token.deploy();
  });
  
  describe("Deployment", function () {
    it("Should set the right owner", async function () {
      expect(await hardhatToken.owner()).to.equal(owner.address);
    });
    it("Should assign the total supply of tokens to the owner", async function () {
      const ownerBalance = await hardhatToken.balanceOf(owner.address);
      expect(await hardhatToken.totalSupply()).to.equal(ownerBalance);
    });
  });

  describe("Transactions", function () {
    it("Should trasfer tokens between accounts", async function () {
      //owner account to addr1.address
      await hardhatToken.transfer(addr1.address, 5);
      const addr1Balance = await hardhatToken.balanceOf(addr1.address);
      expect(addr1Balance).to.equal(5);

      await hardhatToken.connect(addr1).transfer(addr2.address, 5);
      const addr2Balance = await hardhatToken.balanceOf(addr2.address);
      expect(addr2Balance).to.equal(5);
    });

    it("Should fail if sender does not have enough tokens", async function () {
      const initialOwnerBalance = await hardhatToken.balanceOf(owner.address); //10000
      await expect(
        hardhatToken.connect(addr1).transfer(owner.address, 1) //initially - 0 tokens addr1
      ).to.be.revertedWith("Not enough tokens");
      expect(await hardhatToken.balanceOf(owner.address)).to.equal(
        initialOwnerBalance
      );
    });

    it("Should update balances after transfers", async function () {
      const initialOwnerBalance = await hardhatToken.balanceOf(owner.address);
      await hardhatToken.transfer(addr1.address, 5);
      await hardhatToken.transfer(addr2.address, 10);

      const finalOwnerBalance = await hardhatToken.balanceOf(owner.address);
      expect(finalOwnerBalance).to.equal(initialOwnerBalance - 15);

      const addr1Balance = await hardhatToken.balanceOf(addr1.address);
      expect(addr1Balance).to.equal(5);
      const addr2Balance = await hardhatToken.balanceOf(addr2.address);
      expect(addr2Balance).to.equal(10);
    });
  });
});
```
# Debugging
for debugging we have to import hardhat inside the .sol file.
`import "hardhat/console.sol";`
- this debugging to used to find the flaws in the code... when we are testing the code by sending values to it some times which testing an error occuring to find that error we used to debugg the functions of .sol file.
- and to know to flow if test case we use debugging.
- For debugging we write js in sol file

### .sol file for debugging
```
//SPDX-License-Identifier: UNLICENSED
pragma solidity >=0.5.0 <0.9.0;

import "hardhat/console.sol";

contract Token {
    string public name = "HardHat Token";
    string public symbol = "HHT";
    uint256 public totalSupply = 10000;

    address public owner;

    mapping(address => uint256) balances;

    constructor() {
        balances[msg.sender] = totalSupply;
        owner = msg.sender;
    }

    function transfer(address to, uint256 amount) external {
        console.log("**Sender balance is %s tokens**", balances[msg.sender]);
        console.log(
            "**Sender is sending %s tokens to %s address**",
            amount,
            to
        );

        require(balances[msg.sender] >= amount, "Not enough tokens");
        balances[msg.sender] -= amount; //balances[msg.sender]=balances[msg.sender]-amount;
        balances[to] += amount;
    }

    function balanceOf(address account) external view returns (uint256) {
        return balances[account];
    }
}
```
test.js file
```
const { expect } = require("chai");

describe("Token Contract", function () {

  let Token;
  let hardhatToken;
  let owner;
  let addr1;
  let addr2;
  let addrs;

  beforeEach(async function () {
    Token = await ethers.getContractFactory("Token");
    [owner, addr1, addr2, ...addrs] = await ethers.getSigners();
    hardhatToken = await Token.deploy();
  });
  
  describe("Deployment", function () {
    it("Should set the right owner", async function () {
      expect(await hardhatToken.owner()).to.equal(owner.address);
    });
    it("Should assign the total supply of tokens to the owner", async function () {
      const ownerBalance = await hardhatToken.balanceOf(owner.address);
      expect(await hardhatToken.totalSupply()).to.equal(ownerBalance);
    });
  });

  describe("Transactions", function () {
    it("Should trasfer tokens between accounts", async function () {
      //owner account to addr1.address
      await hardhatToken.transfer(addr1.address, 5);
      const addr1Balance = await hardhatToken.balanceOf(addr1.address);
      expect(addr1Balance).to.equal(5);

      await hardhatToken.connect(addr1).transfer(addr2.address, 5);
      const addr2Balance = await hardhatToken.balanceOf(addr2.address);
      expect(addr2Balance).to.equal(5);
    });

    it("Should fail if sender does not have enough tokens", async function () {
      const initialOwnerBalance = await hardhatToken.balanceOf(owner.address); //10000
      await expect(
        hardhatToken.connect(addr1).transfer(owner.address, 1) //initially - 0 tokens addr1
      ).to.be.revertedWith("Not enough tokens");
      expect(await hardhatToken.balanceOf(owner.address)).to.equal(
        initialOwnerBalance
      );
    });

    it("Should update balances after transfers", async function () {
      const initialOwnerBalance = await hardhatToken.balanceOf(owner.address);
      await hardhatToken.transfer(addr1.address, 5);
      await hardhatToken.transfer(addr2.address, 10);

      const finalOwnerBalance = await hardhatToken.balanceOf(owner.address);
      expect(finalOwnerBalance).to.equal(initialOwnerBalance - 15);

      const addr1Balance = await hardhatToken.balanceOf(addr1.address);
      expect(addr1Balance).to.equal(5);
      const addr2Balance = await hardhatToken.balanceOf(addr2.address);
      expect(addr2Balance).to.equal(10);
    });
  });
});





// ---------------------------------------


// const{expect} = require("chai"); // for importing chai
// describe is from macha. by this we are creating a framework. function will increase the visibilty for testing.
// expect is from chai library.

// describe("Token contract", function(){
//     it("Deployment should assign the total supply of tokens to the owner", async function(){
//         const [owner] = await ethers.getSigners();

//         const Token = await ethers.getContractFactory("Token"); //instance contract

//         const hardhatToken = await Token.deploy();//deploy contract        

//         const ownerBalance = await hardhatToken.balanceOf(owner.address);//ownerBalance = 10000
       
//         expect(await hardhatToken.totalSupply()).to.equal(ownerBalance);//totalSupply = 10000
//     })

//     it("Should transfer tokens between accounts", async function(){
//         const [owner, addr1, addr2] = await ethers.getSigners();

//         const Token = await ethers.getContractFactory("Token"); //instance contract

//         const hardhatToken = await Token.deploy();//deploy contract

//         //Transder 100 tokens from owner to add1
//         await hardhatToken.transfer(addr1.address, 10);
//         expect(await hardhatToken.balanceOf(addr1.address)).to.equal(10);

//         //transfer 5 Tokens from addr1 to addr2
//         await hardhatToken.connect(addr1).transfer(addr2.address, 5);
//         expect(await hardhatToken.balanceOf(addr2.address)).to.equal(5);
//     })
// })
```

for test command is 
```
npx hardhat test
```

# Deployment in hardhat network testnet using hardhat
-For deployment create an deploy.js file inside the scripts folder
### code for deployment in hardhat network
```
async function main() {
    const [deployer] = await ethers.getSigners();
  
    console.log("Deploying contracts with the account:", deployer.address);
  
    const token = await ethers.deployContract("Token");
  
    console.log("Token address:", await token.getAddress());
  }
  
  main()
    .then(() => process.exit(0))
    .catch((error) => {
      console.error(error);
      process.exit(1);
    });
```
- To deploy testnet we have to make our laptop as a node in network, but its very difficult. So infura or alchemy are going to help us for that we have to create a api key on alchemy and use that api in our hardhat.config.js file along with our metamask private key.

### code for deployment in test net

--hatdhat.config.js
```
require("@nomicfoundation/hardhat-toolbox");

// Go to https://infura.io, sign up, create a new API key
// in its dashboard, and replace "KEY" with it
const INFURA_API_KEY = "KEY";

// Replace this private key with your Sepolia account private key
// To export your private key from Coinbase Wallet, go to
// Settings > Developer Settings > Show private key
// To export your private key from Metamask, open Metamask and
// go to Account Details > Export Private Key
// Beware: NEVER put real Ether into testing accounts
const SEPOLIA_PRIVATE_KEY = "YOUR SEPOLIA PRIVATE KEY";

module.exports = {
  solidity: "0.8.9",
  networks: {
    sepolia: {
      url: `https://sepolia.infura.io/v3/${INFURA_API_KEY}`,
      accounts: [SEPOLIA_PRIVATE_KEY]
    }
  }
};
```
--deploy.js
```
async function main() {
    const [deployer] = await ethers.getSigners();
  
    console.log("Deploying contracts with the account:", deployer.address);
  
    const token = await ethers.deployContract("Token");
  
    console.log("Token address:", await token.getAddress());
  }
  
  main()
    .then(() => process.exit(0))
    .catch((error) => {
      console.error(error);
      process.exit(1);
    });
```



