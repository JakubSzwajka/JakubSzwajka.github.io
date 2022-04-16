---
layout: post
title: Ethereum ICO - The token and crowdsale
published: false
comments: true
excerpt_separator: <!--more-->
---

One year passed since the very first post here. And half from the last one xd. Time to get cracking!
Last year was huge in the crypto space. I sank into it too. The more I'm learning (losing money), the more I'am into it. So here we are learning solidity and having fun! 

The starting point is always the same: some tutorial. Let's give some kudos to [Dapp Univeristy](https://www.youtube.com/channel/UCY0xL8V6NzzFcwzHCgB8orQ) for making a lot of them! I'm following [real world ico](https://www.youtube.com/watch?v=ir-IRmMTG4Q&list=PLS5SEs8ZftgULF-lbxy-is9x_7mTMHFIN&index=2) tutorial but this part is only about creating Token with a few extra changes.

let's start!

oh.. one more. My code is [here](https://github.com/JakubSzwajka/ethereum_ico)!


## ICO? 

A little bit of theory. ICO (Initial Coin offering) is the way of raising capital by companies. You give them money, they give you tokens. And that is basicaly it. So the goal is to create some token for you and let you pay me for it with BTC for example. Whoooa! 

## Some basics 

Since this is my first solidity project, I think it is worth to wrote down some basic commands and terms to be remembered. 

**Frameworks to make life easier. [Truffle](https://trufflesuite.com/)**

```bash
npm install -g truffle
```

Then you can check your truffle version with: ``truffle version`` command. It will print some more information witch is quite useful. 

```txt
Truffle v5.5.7 (core: 5.5.7)
Ganache v^7.0.3
Solidity v0.5.16 (solc-js)
Node v17.8.0
Web3.js v1.5.3
```
To initialize project structure run: ``truffle init`` 

**You need some ethereum based blockchain on your machine and cli for it. It would be nice to test everything locally.**

```bash
npm install -g ganache-cli
```

To run blockchain with ganache: ``ganache-cli``. Bum! blockchain running!

Extra note: 

I had some problems while migrating my contracts to ganache. I couldn't connect with it. Running ganache-cli with specified port and host was helpful. Consider using ``ganache-cli -h localhost -p 8545``.

## The token

Thanks to openzeppelin-solidity, creating token based on ERC20 is just inheriting from ERC20 class. Mine will be called BonkToken. Don't ask me why :) 

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.13;

import "openzeppelin-solidity/contracts/token/ERC20/ERC20.sol";

contract BonkToken is ERC20{
    constructor(string memory _name, string memory _symbol) 
    ERC20(_name, _symbol)
    {

    }

}
```

And that is basicaly it ;) You can inherit from other classes to add functionalities and other properties to your token. Now run ``truffle compile`` to check if everything is ok!

## Migration 

Migration is basicly specifying how your contract should be deployed into the network. Network params are specified in truffle-config.js file. 

```solidity
const BonkToken = artifacts.require("./BonkToken.sol");

module.exports = function (deployer) {
    const _name = "Bonk Token";
    const _symbol = "BNK";
    deployer.deploy(BonkToken, _name, _symbol);
};
```
After defining migration, run ``truffle migrate``. 

## Tests 

Since I'm new to solidity and JS too actually, I was writing my tests based on [openzeppelin mostly](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/test/token/ERC20/ERC20.test.js) and some tutorial tips.

So the basic test for name and symbol of our token should look like:

```js
const BonkToken = artifacts.require('BonkToken');
const { expect } = require('chai')

contract('BonkToken', accounts => {

    const _name = 'Bonk Token';
    const _symbol = 'BNK'; 

    describe('token attributes', function () {

        beforeEach(async function () {
            this.token = await BonkToken.new(_name, _symbol, _decimals); 
        })

        it('has the correct name', async function () {
            expect(await this.token.name()).to.equal(_name)
        })
        
        it('has the correct symbol', async function () {
            expect(await this.token.symbol()).to.equal(_symbol)
        })
    })

})

```

To run those tests, make sure you have Ganache running (``ganache-cli ``) and run ``truffle test``. Result should look like: 

```bash
  Contract: BonkToken
    token attributes
✓ Transaction submitted successfully. Hash: 0xb04baef19503e3c4ff529e857b28045d50eb1461048a85d3f08686059f4a8318
      ✓ has the correct name
✓ Transaction submitted successfully. Hash: 0xd75e5bf01f7e3876f246a454c5fff3263f584a285f06f07b4c57896127313897
      ✓ has the correct symbol

  2 passing (269ms)
```

## Extending token 

So just for practice I wanted to extend my token with decimals field as it is in tutorial. The difference is that I'm inheriting from ERC20 which has a only symbol and name parameters in constructor. ERC20 by default returns 18 for decimals. 

So first things first, I've added tests which ofc failed. Extended test file should look like: 

```js
const BonkToken = artifacts.require('BonkToken');
const { BN } = require('@openzeppelin/test-helpers')
const { expect } = require('chai')

contract('BonkToken', accounts => {

    const _name = 'Bonk Token';
    const _symbol = 'BNK'; 
    const _decimals = new BN(18); 

    describe('token attributes', function () {

        beforeEach(async function () {
            this.token = await BonkToken.new(_name, _symbol, _decimals); 
        })

        it('has the correct name', async function () {
            expect(await this.token.name()).to.equal(_name)
        })
        
        it('has the correct symbol', async function () {
            expect(await this.token.symbol()).to.equal(_symbol)
        })
        
        it('has the correct decimals', async function () {
            expect(await this.token.decimals()).to.be.bignumber.equal(_decimals)
        })
    })

})
```

The next thing is to make some changes in BonkToken itself. Updated file: 

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.13;

import "openzeppelin-solidity/contracts/token/ERC20/ERC20.sol";

contract BonkToken is ERC20{

    uint8 private _decimals; 

    constructor(string memory _name, string memory _symbol, uint8 decimals_) 
    ERC20(_name, _symbol)
    {
        _decimals = decimals_;
    }

    function decimals() public override view returns(uint8){
        return _decimals;
    }
}
```
* I've added new parameter for BonkToken in constructor which will set contract property. Easy peasy
* In [ERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol), function decimals returns 18 by default. Since I wanted to have it parametrized (ofc. I will leave it 18 xd), I've simply override decimals function with returning _decimals parameter. 


## Deploy token to network

To get access to truffle console, run ``truffle console``

Then in console:
```bash
truffle(development)> BonkToken.deployed( ).then((t) => {token = t})
undefined
truffle(development)> token.address
'0x14213DC7023Daf74e2D063d305978b5bdadB3beD'
truffle(development)> token.name( )
'Bonk Token'
truffle(development)> token.symbol( )
'BNK'
truffle(development)> token.decimals( )
BN { negative: 0, words: [ 18, <1 empty item> ], length: 1, red: null }
```

# Crowdsale 

So we have a token and some tests to confirm that it works. We know how to deploy it to local Ethereum chain too. Let's talk about a Crowdsale contract. 

I'll keep this Crowdsale simple here. The main properties will be: 

* BonkToken will be minted,

* There will be hard cap for this Crowdsale,

* Min amount of eth to invest will be defined. We don't want to have "penny investors". 


## The Crowdsale 

Creating basic crowdsale follows exact the same steps as making token. Inheritance here is a blessing when creating new thinks quickly in Solidity (one week Solidity dev opinion alert :p ).

```solidity
pragma solidity 0.5.5;

contract BonkTokenCrowdsale is Crowdsale {
   constructor(
        uint256 _rate, 
        address payable _wallet, 
        IERC20 _token,
    )
    Crowdsale(_rate, _wallet, _token)
    public 
    {
    }

}

```
There is more magic to make in tests. Within new file lets test creation of Crowdsale with proper token, rate to eth, and wallet address to raise some funds. 

```js 
contract('BonkTokenCrowdsale', function ([_, wallet, investor_1, investor_2]) {

    beforeEach(async function () {

        this.name = 'BonkToken';
        this.symbol = 'BNK';
        this.decimals = new BN(18);

        this.token = await BonkToken.new(
            this.name,
            this.symbol,
            this.decimals
        );


        this.rate = new BN(1000);
        this.wallet = wallet;

        this.crowdsale = await BonkTokenCrowdsale.new(
            this.rate,
            this.wallet,
            this.token.address,
        );
        
    });

    describe('crowdsale', function () {
        
        it('tracks the rate', async function () {
            expect(await this.crowdsale.rate()).
                to.be.bignumber.equal(this.rate);
        });
        
        it('tracks the wallet', async function () {
            expect(await this.crowdsale.wallet()).
                to.equal(this.wallet);
        });
        
        it('tracks the token', async function () {
            expect(await this.crowdsale.token()).
                to.equal(this.token.address);
        });
    });

    describe('accepting payments', function () {
        it('should accept payments', async function () {
            await this.crowdsale.sendTransaction({ value: ether("1"), from: investor_1 }).should.be.fulfilled; 
            await this.crowdsale.buyTokens(investor_1, { value: ether("1"), from: investor_2 }).should.be.fulfilled;
        });
    });

``` 

So what is actually happening here: 

* Create a new BonkToken and create Crowdsale with it. The rate is set to 1000, which means that 1000 BNK = 1 ETH. 

* Next three checks are only about passing proper parameters to the constructor. I'll not dive into details :p 

* The last is about possibily of makeing a trasaction. TO FIX 

## Minted Crowdsale 

As I said previously: next feature, next inheritance. Let's add MintedCrowdsale to BonkTokenCrowdsale. When diving into MintedCrowdsale contract, there is only _deliverToken( ) method. No need to consider any constructor changes. 

```solidity
pragma solidity 0.5.5;

contract BonkTokenCrowdsale is Crowdsale, MintedCrowdsale {
   constructor(
        uint256 _rate, 
        address payable _wallet, 
        IERC20 _token,
    )
    Crowdsale(_rate, _wallet, _token)
    public 
    {
    }

}

```

The thing to change is the BonkToken. Since the Crowdsale will mint our token it should be owned by it and mintable. Here comes the Ownable contract and ERC20Mintable. 

Ownable is about: 

* 

and the ERC20Mintable adds: 

* 

The token looks like this after changes: 

```solidity 
// SPDX-License-Identifier: MIT
pragma solidity 0.5.5;

import "@openzeppelin/contracts/token/ERC20/ERC20Detailed.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20Mintable.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20Pausable.sol";
import "@openzeppelin/contracts/ownership/Ownable.sol";

contract BonkToken is ERC20Mintable, ERC20Detailed, Ownable{

    constructor(string memory _name, string memory _symbol, uint8 _decimals) 
        ERC20Detailed(_name, _symbol, _decimals)
        public
    {

    }
}
``` 

Before we start testing Minting token by Crowdsale we need to transfer ownership of the token, to newly created Crowdsale with following functions in the beforeEach function: 

```js
// create token 
... 

// create Crowdsale 
... 

await this.token.addMinter(this.crowdsale.address);
await this.token.transferOwnership(this.crowdsale.address);
```

Since then we can add some tests for minting. 

```js 
    describe('minted crowdsale', function () {
        it('mints token after purchase', async function () {
            const originalTotalSupply = await this.token.totalSupply();
            await this.crowdsale.sendTransaction({ value: ether("1"), from: investor_1 });
            const newTotalSupply = await this.token.totalSupply();

            expect(newTotalSupply > originalTotalSupply).to.be.true;
        });
    });

``` 
This test is about minting new tokens. On every transaction made in Crowdsale, total supply of BonkTokens should increase. 

## Capped Crowdsale 



```solidity


```
