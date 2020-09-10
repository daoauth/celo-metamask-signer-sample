<template>
  <div class="container is-fluid">
    <div class="card">
      <header class="card-header">
        <p class="card-header-title">
          <span class="icon">
            <img src="/celo-metamask-signer-sample/icon-celo-CELO-color-f.svg">
          </span>
          &nbsp;
          Celo MetaMask Signer
        </p>
      </header>
      <div class="card-content">
        <div class="control">
          <div class="select">
            <select>
              <option selected>
                https://alfajores-forno.celo-testnet.org
              </option>
            </select>
          </div>
        </div>
        <br>
        <div class="field">
          <p class="control has-icons-left">
            <input v-model="account" class="input" type="text" readonly>
            <span class="icon is-small is-left">
              <i class="fas fa-wallet" />
            </span>
          </p>
        </div>
        <div v-for="item in balance" :key="item.name" class="field">
          <p class="control has-icons-left">
            <input v-model="item.value" class="input" type="text" readonly>
            <span class="icon is-small is-left">
              <img :src="item.icon">
            </span>
          </p>
        </div>
        <a
          :href="linkAddress"
          target="_blank"
          rel="noopener noreferrer"
          class="button--grey"
        >
          Blockscout - Address
        </a>
        <hr class="dashed">
        <div class="field is-horizontal">
          <div class="field-body">
            <div class="field is-expanded">
              <div class="field has-addons">
                <p class="control">
                  <a class="button is-static">
                    To
                  </a>
                </p>
                <p class="control is-expanded">
                  <input v-model="to" class="input" type="text">
                </p>
              </div>
            </div>
          </div>
        </div>
        <div class="field is-horizontal">
          <div class="field-body">
            <div class="field is-expanded">
              <div class="field has-addons">
                <p class="control">
                  <a class="button is-static">
                    Value
                  </a>
                </p>
                <p class="control is-expanded">
                  <input v-model="value" class="input" type="number">
                </p>
              </div>
            </div>
          </div>
        </div>
        <div class="links">
          <a
            v-if="!sended"
            class="button--grey"
            @click="send"
          >
            Send
          </a>
          <a
            v-if="sended && linkTx"
            :href="linkTx"
            target="_blank"
            rel="noopener noreferrer"
            class="button--grey"
          >
            Blockscout - Transaction
          </a>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { sendTransaction } from '@did-kr-cg/celo-metamask-signer'
const ContractKit = require('@celo/contractkit')
const Web3 = require('web3')

export default {
  data () {
    return {
      account: 'loading...',
      web3: null,
      kit: null,
      balance: [
        { name: 'CELO', icon: '/celo-metamask-signer-sample/icon-celo-CELO-dark-f.svg', value: 'loading...' },
        { name: 'cUSD', icon: '/celo-metamask-signer-sample/icon-celo-dollar-black-f.svg', value: 'loading...' }
      ],
      to: '',
      value: '',
      sended: false,
      linkAddress: null,
      linkTx: null
    }
  },

  async mounted () {
    if (typeof window.ethereum !== 'undefined') {
      await window.ethereum.enable()
      this.web3 = new Web3(window.ethereum)
    } else {
      return
    }
    const accounts = await this.web3.eth.getAccounts()
    this.account = accounts[0]

    this.kit = ContractKit.newKit('https://alfajores-forno.celo-testnet.org')
    const totalBalance = await this.kit.getTotalBalance(this.account)
    for (let i = 0; i < this.balance.length; i++) {
      this.balance[i].value = this.kit.web3.utils.fromWei(totalBalance[this.balance[i].name].toString())
    }
    // this.balance.push({ name: 'Locked', value: this.kit.web3.utils.fromWei(totalBalance.lockedCELO.toString()) })
    // this.balance.push({ name: 'Panding', value: this.kit.web3.utils.fromWei(totalBalance.pending.toString()) })

    this.linkAddress = `https://alfajores-blockscout.celo-testnet.org/address/${this.account}`
  },

  methods: {
    async send () {
      const transactionParameters = {
        from: this.account,
        to: this.to,
        value: this.value
      }
      this.sended = true
      const tx = await sendTransaction(this.kit, this.web3, transactionParameters)
      const receipt = await tx.waitReceipt()
      this.linkTx = `https://alfajores-blockscout.celo-testnet.org/tx/${receipt.transactionHash}`
      const totalBalance = await this.kit.getTotalBalance(this.account)
      for (let i = 0; i < this.balance.length; i++) {
        this.balance[i].value = this.kit.web3.utils.fromWei(totalBalance[this.balance[i].name].toString())
      }
    }
  }
}
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family:
    'Quicksand',
    'Source Sans Pro',
    -apple-system,
    BlinkMacSystemFont,
    'Segoe UI',
    Roboto,
    'Helvetica Neue',
    Arial,
    sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>
