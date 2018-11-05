---
title: "Anatomy of an Ethereum dApp with Realtime Replication using Zeppelin, Python, Postgresql, React and Geth"
#title: How Did We Replicate Our Smartcontract Transactions In Almost Realtime With Our Ethereum Node
date: 2018-03-13T20:26:52Z
draft: false
---

We got many questions on how [CityMayor](https://citymayor.co), our **dApp to trade cities with Ethereum**, was working under the hood. We will try to explain it in this blog post and the reasons behind those decisions.

For our requirements:

- We wanted to offer the **best dApp experience** to the [CityMayor](https://citymayor.co) users and avoid a slow dApp.
- Non-ethereum users should to be able to take a look at what we were doing without the need to install metamask or use something like Mist/MetaMask. This also means that as an ethereum user you can check what's up when you're not on your main Ethereum device.
- Help people nderstand what was happening without having to check for their transactions manually in Metamask or Etherscan
- Make it as simple as possible for us (developers) to add and maintain features, but also make it easy to deal with the smart contract.

The first solution we thought about was to handle everything in the dApp. It would have 2 distinct parts. The smart contract, running on the blockchain, and the frontend that would take care of handling all the data processing to the user. However, even if we keep the data structure in the [CityMayor contract source](https://etherscan.io/address/0x4bdde1e9fbaef2579dd63e2abbf0be445ab93f10#code) as simple as possible, we didn't want to handle the data processing in the frontend. The other con, was that we couldn’t really understand what happens to some users if all the data were in the frontend, what would have made the debuguing process a lot harder.

<!-- Need to find better transition -->
At the end, we decided to have an architecture with 4 distinct parts, that will allow us to **easily interract with the data, debug, and fix things quickly**:

- The smart contract
- A listener
- An API
- The dApp

Let's have a deeper look on those 4 separate parts, with the tools/technologies we used to build them.

## 1. The (Smart) Contract
The smart contract is the core of [CityMayor](https://citymayor.co). It is the only place where  we defined all the rules for every transactions.

We use [Zeppelin](https://github.com/OpenZeppelin/zeppelin-solidity), an open-source framework, to help write a more secure contract really quickly. If you also want to create a smartcontract, we strongly recommend you to use it. 

There is multiple ways to interract with our contract:

- calling it **via our dApp**
- calling it **directly** (with geth, mist, metmask)

Wait, what do you mean by "_directly_"? 

You can actually interract with any Ethereum contracts by calling any function of it directly. For this, you just need an Ethereum wallet like Mist or MetAmask, know the address of the contract, and the name of the function. For example for CityMayor, we have [open sourced]((https://etherscan.io/address/0x4bdde1e9fbaef2579dd63e2abbf0be445ab93f10#code)) our contract, so simply calling any public method name like `buyCity` or `sellCityForEther` would work without having to open any website!
![Ethereum Smart Contract Direct Interraction](/img/architecture/direct-interraction.png)


However, the interest and feeling of owning a digital asset would not have been the same. In our case, the dApp helps the user to represent cities in the Ethereum world, and make the experience of using Ethereum more fun and friendly!

![Ethereum Smart Contract Direct Interraction](/img/architecture/via-dapp-interraction.png)


## 2. The dApp (or Decentralized Application)
We built the dApp on top of ReactJS with the help of [web3js library](https://github.com/ethereum/web3.js/). 

We picked [React JS](https://reactjs.org/) and [Redux](https://redux.js.org/) mainly because one of our developer is used to it and could build the app really quickly with it. Some cool features like the [Redux Store](https://redux.js.org/api-reference/store) helped us to make the user experience smoother. 

<!-- The most challenging part of this choice was for the 2 others developers that were not really familiar with it to pick it up. An alternative would have been using [VueJS](https://vuejs.org/), but every developer has its own preference. -->


What can be confusing with creating a dApp for Ethereum, is that **you don't need to setup the web3js library** in your application. Your users have to do it, with a provider (like Metamask) that will inject the library in your website. You'll then use all the features of web3 to call the functions of your smart contract.

A good practice is to detect if we can access to the web3 object. If so, we allow the user to do all the actions in our dApp. Otherwise, we recommend them one provider like [Metamask](https://metamask.io)(that will inject the web3js library. A bad practice we ofren see is to forbid your potencial users to discover your dApp if they don't have any provider setup in their browser.

![Asking user to setup Metamaks](/img/architecture/asking-metamask.png)



So our dApp interracts with the contract with the help of a provider like Metamask. When the user wants to do a transaction on our contract, we call a function in web3JS that will open a Metamask popup. This popup will be pre-filled with the function you ask to call, and if necessary, the amount of Ether you need to send to the contract.

![Asking user to setup Metamaks](/img/architecture/daap-interractions.png)

After the user confirm the action, you'll need to handle all the cases that could happen, like pending, failed or confirmed status. The transaction can fail for multiple reasons. We will talk more about those reasons and how to handle them in an other post. 


Perhaps now you're wondering if the data we show on the dApp come from the blockchain? It's not. The react app uses a traditionnal API to show all the informations to our users. Those informations are stored in a Postgresql database. We replicate all those datas from the Ethereum blockchain to the SQL database. And this, with the help of our **listener**.

<!-- This choice was made to be able to do data manipulation easier, and have a reliable  -->


## 3. The Listener (with near Realtime replication)
The listener is certainly the main component that help us (developers) to **ship new features faster**. It is also easier for us to deal with a traditional API than raw datas coming from the blockchain. 

To achieve **near real time replication**, we use the help of [events](http://solidity.readthedocs.io/en/latest/contracts.html#events). Each information changing in our smart contract trigger some events. A **listener** catches the change and replicate instantly the data in the database. We buit the listener with Python and the [web3 py](https://github.com/ethereum/web3.py) library.

![event in citymayor](/img/architecture/event-triggered.png)

We made this listener as solid as possible and capable of handling any kind of unexptected scenarios. It also has a "_backfill mode_" in case where we would need to reprocess some data for any reasons. We use the Ethereum blockchain as a **single source of truth**.

The listener also has the ability to recover in case of crash. We did that with having a checkpoint at every new event received. This checkpoint is kept in a text file.

The listener has to read the events from an Ethereum node. At first, we ran our own Ethereum node, with <a href="https://github.com/ethereum/go-ethereum/">geth</a>. However, maintaining an Ethereum node wasn't the easiest thing to do. It took us lot of maintenance, often crashed, needed lot of SSD disk space. It became really costly (for time and money) to maintain it. We decided to switch to <a href="https://nodablock.com">Nodablock</a> to access to the Ethereum network. They will provide an API and access to the events. It is perfect for the listener, and allow is to focus on the dApp without having to worry if our listenner had access to the Ethereum network.


![realtime ethereum replication](/img/architecture/listener.png)

## 4. Serving the API to the dApp

The main reason to have the API was to make easy data manipulation.

We use Python with the [Flask framework](http://flask.pocoo.org/) to serve our API. We love Python because any developers can jump in the code and be productive really quickly. We're also excited to release soon a golang websocket server that will be used for the live stream journal!

![realtime ethereum replication](/img/architecture/live-journal.png)

The other reason to have an API was to be able to store sensible informations. As all datas stored in the Ethereum blockchain are  public, we couldn't use it as the only data store. This was really useful for us to store emails of our users. We will cover how to do a transaction with your own API without having to go though the blockchain in an other blog post.

Here a schema of the CityMayor database (old version) with next to it the data structure of the contract representing it.
[![realtime ethereum replication](/img/architecture/bdd-diagram.png)](/img/architecture/bdd-diagram.png)
 The representation we chose was good enough for our use case, I'm sure other representations with better abstrations could have been made.


## Conclusion 
- The CityMayor dApp is an hybrid one: half centralized, half decentralized. This mainly helps us to ship and maintain features easily
- All key transactions (buy/sell/offers) happen directly on the Ethereum blockchain with the help of an Ethereum provider (Metamask, Mist, etc..)
- We have a "_listener_", helping to replicate in (almost) realtime all data changes that happen on our contract, from the Ethereum network to a PostgreSQL database. 
- The dApp only read data from a REST API


We hope you enjoyed this blog post! Make sure to have a look in the [Citymayor dApp](https://citymayor.co), and start to trade some cities! Don't forget to subscribe to our newsletter to learn more about blockchain or our venture! We will soon blog about the 4 parts of our architecture in details and our different experiments to acquire new users.

Stay tuned!

<i class="fas fa-university"></i> **The United Nations**