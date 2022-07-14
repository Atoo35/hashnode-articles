## Fetching Mirror Posts

Hello everyone 👋, I, Atharva Deshpande am back with another article. But this time we move away from discord and into blockchain and [Arweave](https://www.arweave.org/) and [Mirror](https://mirror.xyz/).

In this article, we would understand how to fetch mirror posts of any Ethereum address by using Arweave's Graph. This article expects prior knowledge of the fundamentals of blockchain,  [Arweave](https://www.arweave.org/) and [Mirror](https://mirror.xyz/).

Here is the link to [Github Repository](https://github.com/Atoo35/fetch-mirror-posts). You can go ahead and get the code directly but I would recommend reading through the article once.


## **Prerequisites**

- Familiarity with blockchain,  [Arweave](https://www.arweave.org/) and [Mirror](https://mirror.xyz/).

- Any code editor. ( I would be using [VSCode](https://code.visualstudio.com/download) )

- Basic understanding of Javascript

- Machine with [NodeJS](https://nodejs.org/en/)(v16.15.0) installed.


## **Approach** 🧠

The process of retrieving Mirror posts is split into 2 steps.
- Step 1: Retrieve the transactions of a user from the Arweave graph with the tag name `MirrorXYZ`.
- Step 2: Query the Arweave API  `https://arweave.net/<txId>` by passing the transaction ID to get the JSON data format.


## **Let's BUIDL It** 🛠️

1. Create a folder, open it in VSCode and run the command
```js
npm init -y
```

    Edit the `package.json` file so that the scripts tag looks like this.
    ```json
    "scripts": {
        "start": "node index.js"
      }
    ```

2. Create an `index.js` file and run the command to install Axios which would be used to query the graph and get the JSON file as well.
```js
npm i axios
```

    Your folder structure should look like this

    ![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1657790855601/1mYeTV1QS.png align="left")


3. Let's write the code to query the Arweave graph to get transactions of a user. In the `index.js` file, copy paste the following code.
```javascript
const axios = require('axios');
arweaveURL = "https://arweave.net";
const fetchTransactions = async (address) => {
  try {
    const result = await axios.post(
      `${arweaveURL}/graphql`,
      {
        query: `
                {
                  transactions(
                      tags:[
                        {
                          name:"App-Name",
                          values:["MirrorXYZ"],
                        },
                        {
                          name:"Contributor",
                          values:["${address}"]
                        }
                      ]
                  sort:HEIGHT_DESC) {
                      edges {
                          node {
                              id
                          }
                      }
                  }
              }
              `
         },
         {
           headers: {
             'Content-Type': 'application/json'
           }
         }
       );
        return result.data.data
      } catch (err) {
        console.log(err);
      }
}
const main = async () => {
     const response = await fetchTransactions("0x9651B2a7Aa9ed9635cE896a1Af1a7d6294d5e902")
     console.log(response.transactions.edges)
}
main()
  .then(() => process.exit(0))
```

    Here, the function `fetchTransactions` simply queries the Arweave Graph using axios. We are sending a graph query that searches for transactions with the tags `App-Name` as `MirrorXYZ` and `Contributor` address as `0x9651B2a7Aa9ed9635cE896a1Af1a7d6294d5e902`.
> Note the above address is of [Mirror Development](https://dev.mirror.xyz/)

    Run the script by typing the following in your terminal.
    ```js
npm start
```
    Your terminal should look somewhat like this

    ![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1657792467523/PwSpGcCQU.png align="left")


3. Let's write the final function which basically fetches the json data of the post. Add the below function anywhere before the `main` function
```javascript
const fetchArweaveTransactionData = async (transactionId) => {
      try {
          const response = await axios.get(`https://arweave.net/${transactionId}`)
          return response.data;
      } catch (error) {
          console.log(error)
      }
}
```

  We simply pass on the transaction id from the response of the graph API to get the post. update your `main` function to the following:
```javascript
const main = async () => {
      const response = await fetchTransactions("0x9651B2a7Aa9ed9635cE896a1Af1a7d6294d5e902")
      console.log(response.transactions.edges)
      const post = await fetchArweaveTransactionData(response.transactions.edges[0].node.id)
      console.log(post)
}
```

    > Note: For the sake of this article, I am fetching post for only the first transaction from the response.
    
    The response is pretty huge to display it here with the console version but here is the browser version.

    ![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1657793042082/9eCSaXomb.png align="left")


That's it, folks we have successfully retrieved Mirror posts.


### Support
If you liked my article, consider supporting me with a coffee ☕️ or some crypto ( ₿, ⟠, etc) 

Here is my public address `0x7935468Da117590bA75d8EfD180cC5594aeC1582`
<a href="https://www.buymeacoffee.com/atoo" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-yellow.png" alt="Buy Me A Coffee" height="41" width="174"></a>

### Lets connect
[Github](https://github.com/Atoo35)

[LinkedIn](https://www.linkedin.com/in/atharva-deshpande-187969140/)

[Twitter](https://twitter.com/atharva_35)

### Feedback
Let me know if I have missed something or provided the wrong info. It helps me keep genuine content and learn as well.
