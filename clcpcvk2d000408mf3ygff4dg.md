# Ethernaut Challenge Level 1

Hey everyone, here is the second article of my series on Ethernaut journey. It's going well for now at least hahaha.

> NOTE: You don't need to read through the entire article if you are looking for some solution to your problem. Head down to the challenges section and see if I encountered the same challenge as you***.***

# Introduction

I found this particular challenge a little easy as I knew one or two things about fallback functions like `receive()`. Well, basically this function is called from the EVM when there is for empty calldata with some ETH has been sent to the contract.

# Let's Start

The task here was to either claim ownership of the contract or successfully withdraw ETH from the contract. Obviously, when you want to claim ownership of a solidity contract, the first thing you look into is the usage of `owner` property and look for places where it has been updated after deployment.

This property was updated in two places, one is the `contribute` function and the other in the `receive` function

```solidity
function contribute() public payable {
    require(msg.value < 0.001 ether);
    contributions[msg.sender] += msg.value;
    if(contributions[msg.sender] > contributions[owner]) {
      owner = msg.sender;
    }
  }
```

```solidity
receive() external payable {
    require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender;
  }
```

The first thought in my mind was to just transfer enough ether so that my contribution becomes greater than the owner as the `contribute` function renounces the ownership of that logic. Well, it turns out that in the constructor, the contribution of the owner is set to 1000 Ether. I am not that rich even on the Goerli testnet hahaha.

Anyway, I then saw the `receive` function and it became very clear what was needed to be done to claim ownership.

1. Contribute some eth using the `contribute` function, keeping in mind the constraint on `value` in the function i.e. I should send &lt;0.001 ether.
    

1. Invoke the receive function and voila.
    
2. Of course, call the `withdraw` function to get my money back haha.
    

How to call the contribute function you ask?

```javascript
await contract.contribute({value:toWei("0.001")})
```

The function expects some eth to be sent, and the above call specifies how to. You might be wondering about the usage of `toWei` function and where even in the contract is it defined. Actually, this is not a function of the contract but rather a function provided on the console by openzeppelin.

Tip: Type `help()` to know what other functions are available.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1673217857602/cef8a9dd-4ebf-41a0-892d-0de5f962adc4.png align="center")

After a successful transaction, it's now time to become the owner of the contract. I called the `send` (or `sendTransaction`) function available directly on EVM and sent some eth again. This time the contract will invoke the `receive` function and voila, I became the OWNERRRRR!!!!.

Below is a screenshot of how to call the function

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1673217927996/4dadb5df-33e5-45f4-aacc-5243bc652263.png align="center")

As I say, never let go of the money, calling the `withdraw` function I got all my eth back.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1673217988522/8b262a48-eec5-42fa-8559-49d7c4ccfcbb.png align="center")

CONGRATULATIONS to me!!!! I successfully hacked the contract.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1673218031787/3ac5b1c8-27fa-454b-a738-431eacf00e76.png align="center")

# Challenges

The challenges I faced while solving were:

1. How to call a function of the contract and send ETH to it?
    
    Solution: In the function, parenthesis send an object with the key as `value` and value as the eth amount in Wei.
    
    > TIP: Use `toWei` to quickly convert to Wei.
    
2. How to invoke the `receive` function of a contract?
    
    Solution: Just use the method `send` or `sendTransaction` with the contract and pass on eth as I did above.
    

# Support

If you liked my article, consider supporting me with a coffee ☕️ or some crypto ( ₿, ⟠, etc)

Here is my public address `0x7935468Da117590bA75d8EfD180cC5594aeC1582`

[![Buy Me A Coffee](https://cdn.buymeacoffee.com/buttons/default-yellow.png align="left")](https://www.buymeacoffee.com/atoo)

### Lets connect

[**Github**](https://github.com/Atoo35)

[**LinkedIn**](https://www.linkedin.com/in/atharva-deshpande-187969140/)

[**Twitter**](https://twitter.com/atharva_35)

### Feedback

Let me know if I have missed something or provided the wrong info. It helps me keep genuine content and learn.