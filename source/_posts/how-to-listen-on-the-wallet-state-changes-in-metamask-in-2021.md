---
title: How to Listen on the Wallet State Changes in MetaMask in 2021
date: 2020-12-25 12:43:05
tags:
---

There are two very common wallet state changes needed to handle: chainId(network) and account changes. It's a very typical pattern to dynamically detect the current active user and chainId at the time of writing any non-trivial application. This article will introduce how to decently listen on these changes.

To manage it, we need to use two API calls:
```
# From https://eips.ethereum.org/EIPS/eip-1193#events
Provider.on('chainChanged', listener: (chainId: string) => void): Provider;
Provider.on('accountsChanged', listener: (accounts: string[]) => void): Provider;
```

Since to connect MetaMask, we can use the ``window.ethereum`` as our ``provider``, so to listen on the state changes, here's a sample code:
```
let that = this
window.ethereum.on('accountsChanged', function (accounts) {
  console.log(`Accounts changed. All connected accounts are: ${accounts}`)
  if (accounts.length == 0) {
    console.log(`All accounts disconnected.`)
  } else {
    that.userAddr = accounts[0] // current active user
    console.log(`Currently the active user is ${accounts[0]}`)
  }
})

window.ethereum.on('chainChanged', (chainId) => {
  console.log(`ChainId changes. Current connected chainId is ${chainId}`)
  if (chainId == correctChainId) {
    console.log(`You are in the correct network.`)
  } else {
    console.log(`You are in the wrong network. Please switch to the network with chainId ${chainId}`)
  }
})
```
Specifically, ``let that = this`` is just a small programming tip, yet very useful if there's a context switch. But definitely you might not need that tip at all, just follow your own application logic!

The ``accountsChanged`` listener can be triggered if the number of connected accounts from 0 to some amount, or from some amount to 0, also it can be triggered if the current active user changes. Whereas the ``chainChanged`` listener can be triggered if the current connected network changes. But just to notice, both the listeners will not be triggered during the loading of the page, which means, if MetaMask has been enabled and some account has been connected to this site, reloading the page will not trigger the listeners.

So to summarize, according to the features of the two listeners introduced above, here I introduce a complete pattern I used in my application:

Once the page is loaded:

1. At the very beginning we need to create two listeners on ``accountsChanged`` and ``chainChanged`` respectively, and in their handlers according to the application logic, I just do a little computations and modified a small amount of meaningful variables, which will be listened on out of the listeners, and there I will do the major computations.(Typically if you use some frameworks, like React, Vue, etc, it can be easily done.)

2. Then we need to check the current active account and the connected chainId and handle the different scenarios.

Besides, if the dapp forces to connect the MetaMask once it's loaded, we can implement the 3rd step just immediate after the above logic, otherwise simply put it in its correct position (mostly it's supposed to be a click handler).

3. Then we deal with the connect logic. I introduce to how connect MetaMask in a previous [article](https://atlasquan.github.io/2020/12/25/how-to-connect-web3-js-to-metamask-in-2021/) in detail. In this step, we only need to handle the logic about connection to MetaMask, and we don't need to know which accounts are finally connected or which network is switched, since we have the separate listeners above to handle such things.
