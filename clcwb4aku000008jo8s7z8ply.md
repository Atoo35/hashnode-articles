# Ethernaut Challenge Level 2

Welcome back to the third article or level 2 of Ethernaut. I had my exams and couldn't find time to write another journey entry, so sorry for that

> NOTE: You don't need to read through the entire article if you are looking for some solution to your problem. Head down to the challenges section and see if I encountered the same challenge as you

# Introduction

Well, this particular level is crazy easy but damn did it eat my entire evening? hahahahah. This one was a slick level and the challenge name couldn't have been more obvious.

# Let's Start

Similar to the last challenge, we have to claim ownership of this smart contract and withdraw all the eth. It is pretty normal for the first instinct to check for assignments of `owner` variable to either address or `msg.sender`.

The only place this variable is being assigned is in the constructor. Well, we cannot do anything with the constructor after the contract has been deployed right? Mehh, nothing to do here, not hackable. But wait, there's a typo in the word `Fallout` in the constructor.

It's written as `Fal1out` and the contract name is `Fallout` . Notice the number 1 is used instead of the alphabet l (L in lowercase). Gotcha, the constructor does not exist for this contract and hence I can call the function `Fal1out` after the deployment.

```solidity
/* constructor */
  function Fal1out() public payable {
    owner = msg.sender;
    allocations[owner] = msg.value;
  }
```

That is exactly what I did, I called the function and passed some eth to it to claim ownership

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1673218195238/9e37e44f-2f9f-440f-8176-eb2f7afc964e.png align="center")

As soon as the transaction went through, I submitted the instance and voila, I was now the owner of the contract. TADAAA!!!!

Don't forget to get your eth back by using the `collectAllocation` function.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1673218380372/384c46fb-806c-45b5-b9f5-ff7bd118fd3d.png align="center")

# Challenges

1. I wasn't able to quickly spot the spelling mistake in the constructor function name.
    
    Solution: First off, clean your eyes and secondly just directly call the so-called constructor lookalike `Fal1out` like a normal function to claim ownership.
    

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