# celo-metamask-signer-sample

## Sample Code
[index.vue](https://github.com/daoauth/celo-metamask-signer-sample/blob/master/pages/index.vue)
```javascript
import { sendTransaction } from '@did-kr-cg/celo-metamask-signer'
const ContractKit = require('@celo/contractkit')
const Web3 = require('web3')

...

async function sendTx () {
  const kit = ContractKit.newKit('https://alfajores-forno.celo-testnet.org')
  let web3 = null

  if (typeof window.ethereum !== 'undefined') {
    await window.ethereum.enable()
    web3 = new Web3(window.ethereum)
  } else {
    return
  }
  const accounts = await this.web3.eth.getAccounts()

  const rawTx = {
    from: accounts[0],
    to: 'TARGET ACCOUNT',
    value: 'VALUE',
    data: 'IF YOU WANT USE SMART CONTRACT'
  }
  const receipt = await sendTransaction(kit, web3, rawTx)
  console.log(`https://alfajores-blockscout.celo-testnet.org/tx/${receipt.transactionHash}`)
}
```

## Build Setup

```bash
# install dependencies
$ yarn install

# serve with hot reload at localhost:3000
$ yarn dev

# build for production and launch server
$ yarn build
$ yarn start

# generate static project
$ yarn generate
```

For detailed explanation on how things work, check out [Nuxt.js docs](https://nuxtjs.org).
