---
layout: post
title: Ethereum ICO - The token and crowdsale
published: false
comments: true
excerpt_separator: <!--more-->
tags: Solidity Web3
---

The previous post was about ERC20 token. Now let's prepare Crowdsale for this token. I'll keep this Crowdsale simple here. The main properties will be: 

* BonkToken will be minted,

* There will be hard cap for this Crowdsale,

* Min amount of eth to invest will be defined. We don't want to have "penny investors". 

<!--more-->

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
