---
layout: post
title: Ethereum ICO - Minted Crowdsale
published: true
comments: true
excerpt_separator: <!--more-->
tags: Web3 Solidity
---

The previous [post](https://jakubszwajka.github.io/Ethereum-token/) was about the ERC20 token. Now let's prepare Crowdsale for this token. I'll keep this Crowdsale simple here.

<!--more-->

## The Crowdsale 

Creating a basic crowdsale follows the same steps as making a Token. Inheritance here is a blessing when creating new things quickly in Solidity (one-week Solidity dev opinion alert 😜).

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
There is more magic to make in tests. Within the new file let's test the creation of Crowdsale with a proper token, rate to eth, and wallet address to raise some funds. 

```js 
contract('BonkTokenCrowdsale', function ([_, wallet, investor_1, investor_2]) {

    beforeEach(async function () {

        this.name = 'BonkToken';
        this.symbol = 'BNK';
        this.decimals = new BN(2);

        this.token = await BonkToken.new(
            this.name,
            this.symbol,
            this.decimals
        );


        this.rate = new BN(10);
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

So what is happening here: 

* Create a new BonkToken and create Crowdsale with it. The rate is set to 10 but what does it mean? 

> How many token units a buyer gets per wei. The rate is the conversion between wei and the smallest and indivisible token unit. So, if you are using a rate of 1 with a ERC20Detailed token with 3 decimals called TOK, 1 wei will give you 1 unit, or 0.001 TOK.

So we are using 16 decimals token in tests, and want to have rate:

 1000 BNK <-> 1 ETH <-> 1000000000000000000 WEI. 
 
The rate 1 will give us 0.0000000000000001 BNK. Since we want to have 1 BNK <-> 0.0001 ETH <-> 1 000 000 000 000 000 WEI we need 10 smalest units of BNK for 1 WEI (0.0000000000000010 BNK <-> 1 WEI ), that means the rate should be 10.

so:

0.0000000000000010 BNK <-> 1 WEI and 1 ETH <-> 1000000000000000000 WEI. Use calculator 🤔. 

> 0.~~00000000000000~~1 * 10000~~00000000000000~~ = 1000


* Next three checks are only about passing proper parameters to the constructor. I'll not dive into details.

* The last is about the possibility of making a transaction. Only checking if all promisses are fulfilled.

## Minted Crowdsale 

As I said previously: next feature, next inheritance. Let's add MintedCrowdsale to BonkTokenCrowdsale. 

When diving into the MintedCrowdsale contract, there is only _deliverToken( ) method. No need to consider any constructor changes. 

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

The thing to change is the BonkToken. Since the Crowdsale will mint our token it should be owned by it and mintable. Here comes the Ownable contract and ERC20Mintable contract. 

Ownable adds access control mechanizm to our token. After inheriting, some functions are restricted to be executed only by Owner.

ERC20Mintable is explaining itself very well in dev docs: 

> Extension of {ERC20} that adds a set of accounts with the {MinterRole}, which have permission to mint (create) new tokens as they see fit. At construction, the deployer of the contract is the only minter.


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

Before we start testing minting the token by Crowdsale we need to transfer ownership of the token, to the newly created Crowdsale with the following functions in the beforeEach function: 

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
This test is about minting new tokens. On every transaction made in Crowdsale, a total supply of BonkTokens should increase. 
