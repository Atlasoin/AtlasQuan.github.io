---
title: How to Connect Web3.js to MetaMask in 2021
date: 2020-12-25 08:50:04
tags:
- Blockchain
---

The blockchain world always evolves so quickly, since it's still a young and energetic field. Without programming with ethereum network for some time, I noticed lots of stuff has changed. This article is inspired by the article *[How to Connect Web3.js to MetaMask in 2020](https://awantoch.medium.com/how-to-connect-web3-js-to-metamask-in-2020-fee2b2edf58a)*. It's worth mentioning that, according to this article, in newer version MetaMask, we can simply use ``window.ethereum`` as the ``provider`` itself, without checking ``window.web3`` for its ``currentProvider``.

However, the implementation mentioned in that article used the code ``window.ethereum.enable();`` which still works until now, but there's an warning:
 >MetaMask: 'ethereum.enable()' is deprecated and may be removed in the future. Please use the 'eth_requestAccounts' RPC method instead.
  For more information, see: https://eips.ethereum.org/EIPS/eip-1102

So that I am not satisfied. Then I planned to write a new 2021 version article to introduce how to manage it without warning. (Even though there's still 5 days remained in 2020 XD but I believe it doesn't matter.)

We should use [request](https://eips.ethereum.org/EIPS/eip-1193#request-1) API instead of ``ethereum.enable()``. (The ``ethereum.send()`` also can work but has been deprecated similarly.) So I recommend using the following code to connect to MetaMask:

```
const Web3 = require("web3");
const connectMetaMask = async () => {
  // If web3 has been initialized somewhere else, we don't need to reconstruct it.
  if (typeof window.web3.currentProvider == 'undefined')  
    window.web3 = new Web3(window.ethereum)
  try {
    let accounts = await ethereum.request({
      method: 'eth_requestAccounts'
    })
    console.log(`Accounts:\n${accounts.join('\n')}`)
    return true
  } catch (e) {
    // Usually it's caused by user self rejecting connection. 
    // All provider error types can be checked here: https://eips.ethereum.org/EIPS/eip-1193#provider-errors
    console.log(e)
    return false
  }
}
```

But before connecting to MetaMask, typically we might want to also check if the browser supports the MetaMask, so we can use the following code to do that:
```
if (typeof window.ethereum == "undefined") {
  if (!(await connectMetaMask())) {
    return
  }
} else {
  console.log(
    `Please install an Ethereum-compatible browser or extension like MetaMask.`
  )
  return
}
```

Definitely you can adjust the code as your own dapp logic!

But just to notice that, this approach is to connect the MetaMask, not to test or listen whether it's connected or not. If you want to listen on the connection changes, please see another of my article *[How to Listen on the Wallet State Changes in MetaMask in 2021](https://atlasquan.github.io/2020/12/25/how-to-listen-on-the-wallet-state-changes-in-metamask-in-2021/)*.