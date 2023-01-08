# Ethernaut Challenge Level 0

Hello everyone, this article is the first one in the series of my Ethernaut Journey. I hope to be regular and reach the end. LFG!!!

> NOTE: You don't need to read through the entire article if you are looking for some solution to your problem. Head down to the challenges section and see if I encountered the same challenge as you.

# Introduction

The first stage is a very very simple one as it's just an introductory challenge to get to understand how the website works and what can a user do. As the text suggests, I tried out some function calls/ object calls like `player`, `getBalance(player)`, etc. I even tried out the `help` function and it gave me some good properties I can play around with.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1673205395217/fd33f24a-6a63-404c-9de6-bcb86d22c246.png align="center")

# Let's Start

As directed I tried out the `await contract.info()` function to begin the challenge, which then asked me to use the `info1()` function. By simply following the instructions on the screen I reached a state where the prompt now talks about authenticating with a password.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1673205725782/b0e45771-0c58-42b0-81ca-8993f3274263.png align="center")

This surely was a bit weird and I wasn't sure what to do, however, with some trial and error I came to know there is a property named password, and by just using the `await contract.password()` function, I was able to get the password and authenticate using the `await contract.authenticate()` function.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1673205892127/78373cf7-7440-439d-b4dd-fa7bfddc2bc4.png align="center")

> NOTE: I got to know about the function `getCleared()` after I submitted the challenge post successful authenticate transaction response.

Once the authentication transaction went through, I went ahead and submitted the instance and all was good!!.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1673206399361/8d97a6f3-d092-4f98-856d-fb9c3b6a9915.png align="center")

# Challenges

1. The step where I had to get the `password` using the property was a bit unclear to me.  
    Solution: Use `await contract.password()` function to get the password.
    
2. After using the `authenticate()` function, I wasn't sure if it was successful or not. I went ahead and submitted the instance to know for sure.  
    Solution: Use `await contract.getCleared()` function to know if you can submit the instance.
    

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