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
          <div class="field-body">
            <div class="field is-expanded">
              <div class="field has-addons">
                <p class="control has-icons-left is-expanded">
                  <input v-model="account" class="input" type="text" readonly>
                  <span class="icon is-small is-left">
                    <i class="fas fa-wallet" />
                  </span>
                </p>
                <p class="control">
                  <a
                    class="button"
                    :href="linkAddress"
                    target="_blank"
                    rel="noopener noreferrer"
                  >
                    <i class="fas fa-search" />
                  </a>
                </p>
              </div>
            </div>
          </div>
        </div>
        <div v-for="item in balance" :key="item.name" class="field">
          <div class="field-body">
            <div class="field is-expanded">
              <div class="field has-addons">
                <p class="control has-icons-left is-expanded">
                  <input v-model="item.value" class="input" type="text" readonly>
                  <span class="icon is-small is-left">
                    <img :src="item.icon">
                  </span>
                </p>
                <p class="control">
                  <a class="button" @click="approve(item.name)">
                    <i class="fas fa-unlock" />
                  </a>
                </p>
              </div>
            </div>
          </div>
        </div>
        <hr class="dashed">
        <div class="tabs is-small is-boxed">
          <ul>
            <li :class="tab===0?'is-active':''" @click="tab=0">
              <a>Transfer</a>
            </li>
            <li :class="tab===1?'is-active':''" @click="tab=1">
              <a>Swap</a>
            </li>
            <li :class="tab===2?'is-active':''" @click="tab=2; getBuyAndSellBuckets();">
              <a>Buckets</a>
            </li>
          </ul>
        </div>
        <div>
          <section v-if="tab===0" class="animated fadeIn faster">
            <label class="label has-text-left is-size-7">Value</label>
            <div class="field">
              <div class="field-body">
                <div class="field is-expanded">
                  <div class="field has-addons">
                    <p class="control has-icons-left is-expanded">
                      <input v-model="tx.value" class="input" type="number">
                      <span class="icon is-small is-left">
                        <img :src="balance[tx.selected].icon">
                      </span>
                    </p>
                    <p class="control">
                      <span class="select" @change="onChangeCurrency">
                        <select>
                          <option v-for="item in balance" :key="item.name" :value="item.name">
                            {{ item.name }}
                          </option>
                        </select>
                      </span>
                    </p>
                  </div>
                </div>
              </div>
              <p class="help has-text-left">
                {{ balance[tx.selected].value }}
              </p>
            </div>

            <label class="label has-text-left is-size-7">To (address)</label>
            <div class="field">
              <div class="field-body">
                <div class="field is-expanded">
                  <div class="field has-addons">
                    <p class="control is-expanded">
                      <input v-model="tx.to" class="input" type="text">
                    </p>
                    <p class="control">
                      <a
                        v-if="!sended"
                        class="button"
                        @click="send"
                      >
                        <i class="fas fa-share" />
                      </a>
                      <a
                        v-if="sended && tx.linkTx"
                        :href="tx.linkTx"
                        target="_blank"
                        rel="noopener noreferrer"
                        class="button"
                      >
                        <i class="fas fa-search" />
                      </a>
                    </p>
                  </div>
                </div>
              </div>
            </div>
          </section>
          <section v-if="tab===1" class="animated fadeIn faster">
            <label class="label has-text-left is-size-7">From ({{ exchange.type }})</label>
            <div class="field">
              <div class="field-body">
                <div class="field is-expanded">
                  <div class="field has-addons">
                    <p class="control has-icons-left is-expanded">
                      <input v-model="exchange.from.value" class="input" type="number">
                      <span class="icon is-small is-left">
                        <img :src="exchange.from.icon">
                      </span>
                    </p>
                    <p class="control">
                      <span class="select" @change="onChangeFrom">
                        <select>
                          <option v-for="item in exchange.buckets" :key="item.name" :value="item.name">
                            {{ item.name }}
                          </option>
                        </select>
                      </span>
                    </p>
                  </div>
                </div>
              </div>
              <p class="help has-text-left">
                {{ balance[exchange.selected].value }} / {{ balance[exchange.selected].approve }}
              </p>
            </div>
            <label class="label has-text-left is-size-7">To (estimated)</label>
            <div class="field">
              <div class="field-body">
                <div class="field is-expanded">
                  <div class="field has-addons">
                    <p class="control has-icons-left is-expanded">
                      <input v-model="exchange.to.value" class="input" type="number" readonly>
                      <span class="icon is-small is-left">
                        <img :src="exchange.to.icon">
                      </span>
                    </p>
                    <p class="control">
                      <a
                        v-if="!sended"
                        class="button"
                        @click="swap"
                      >
                        <i class="fas fa-exchange-alt" />
                      </a>
                      <a
                        v-if="sended && exchange.linkTx"
                        :href="exchange.linkTx"
                        target="_blank"
                        rel="noopener noreferrer"
                        class="button"
                      >
                        <i class="fas fa-search" />
                      </a>
                    </p>
                  </div>
                </div>
              </div>
            </div>
          </section>
          <section v-if="tab===2" class="animated fadeIn faster">
            <div v-for="item in exchange.buckets" :key="item.name" class="field">
              <p class="control has-icons-left">
                <input v-model="item.value" class="input" type="text" readonly>
                <span class="icon is-small is-left">
                  <img :src="item.icon">
                </span>
              </p>
            </div>
          </section>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { sendTransaction } from '@did-kr-cg/celo-metamask-signer'
const ContractKit = require('@celo/contractkit')
const Web3 = require('web3')
const LOADING = 'loading...'

export default {
  data () {
    return {
      tab: 0,
      account: LOADING,
      linkAddress: null,
      web3: null,
      kit: null,
      contracts: {
        goldToken: null,
        stableToken: null,
        exchange: null
      },
      balance: [
        { name: 'CELO', icon: '/celo-metamask-signer-sample/icon-celo-CELO-dark-f.svg', value: LOADING, approve: LOADING },
        { name: 'cUSD', icon: '/celo-metamask-signer-sample/icon-celo-dollar-black-f.svg', value: LOADING, approve: LOADING }
      ],
      tx: {
        to: '',
        value: '',
        selected: 0,
        linkTx: null
      },
      exchange: {
        selected: 0,
        type: 'Sell',
        from: {
          value: 0,
          icon: '/celo-metamask-signer-sample/icon-celo-CELO-dark-f.svg'
        },
        to: {
          value: 0,
          icon: '/celo-metamask-signer-sample/icon-celo-dollar-black-f.svg'
        },
        buckets: [
          { name: 'CELO', icon: '/celo-metamask-signer-sample/icon-celo-CELO-dark-f.svg', value: LOADING },
          { name: 'cUSD', icon: '/celo-metamask-signer-sample/icon-celo-dollar-black-f.svg', value: LOADING }
        ],
        linkTx: null
      },
      sended: false
    }
  },

  watch: {
    'exchange.from.value': {
      handler (value) {
        if (value) {
          if (this.exchange.type === 'Sell') {
            this.getSellTokenAmount()
          } else {
            this.getBuyTokenAmount()
          }
        } else {
          this.exchange.to.value = ''
        }
      },
      deep: true
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
    this.linkAddress = `https://alfajores-blockscout.celo-testnet.org/address/${this.account}`

    this.kit = ContractKit.newKit('https://alfajores-forno.celo-testnet.org')

    this.contracts.goldToken = await this.kit._web3Contracts.getGoldToken()
    this.contracts.stableToken = await this.kit._web3Contracts.getStableToken()
    this.contracts.exchange = await this.kit._web3Contracts.getExchange()

    this.update()
    // this.balance.push({ name: 'Locked', value: this.kit.web3.utils.fromWei(totalBalance.lockedCELO.toString()) })
    // this.balance.push({ name: 'Panding', value: this.kit.web3.utils.fromWei(totalBalance.pending.toString()) })
  },

  methods: {
    async update () {
      const totalBalance = await this.kit.getTotalBalance(this.account)
      for (let i = 0; i < this.balance.length; i++) {
        this.balance[i].value = this.kit.web3.utils.fromWei(totalBalance[this.balance[i].name].toString())
      }
      this.balance[0].approve = LOADING
      this.balance[1].approve = LOADING
      this.balance[0].approve = this.kit.web3.utils.fromWei((await this.allowance(this.contracts.goldToken)))
      this.balance[1].approve = this.kit.web3.utils.fromWei((await this.allowance(this.contracts.stableToken)))
    },

    async sendTx (transactionParameters) {
      try {
        this.sended = true
        const tx = await sendTransaction(this.kit, this.web3, transactionParameters)
        const receipt = await tx.waitReceipt()
        const linkTx = `https://alfajores-blockscout.celo-testnet.org/tx/${receipt.transactionHash}`
        for (let i = 0; i < this.balance.length; i++) {
          this.balance[i].value = LOADING
        }
        this.update()
        // this.sended = false
        return linkTx
      } catch (error) {
        console.log(error)
        this.sended = false
      }
    },

    onChangeCurrency (event) {
      switch (event.target.value) {
        case 'CELO':
          this.tx.selected = 0
          break
        case 'cUSD':
          this.tx.selected = 1
          break
      }
    },

    async send () {
      try {
        if (this.tx.selected === 0) {
          this.tx.linkTx = await this.sendTx({
            from: this.account,
            to: this.tx.to,
            value: this.kit.web3.utils.toWei(this.tx.value)
          })
        } else if (this.tx.selected === 1) {
          this.tx.linkTx = await this.sendTx({
            from: this.account,
            to: this.contracts.stableToken.options.address,
            data: this.contracts.stableToken.methods.transfer(this.tx.to, this.kit.web3.utils.toWei(this.tx.value)).encodeABI()
          })
        }
      } catch (error) {
        console.log(error)
      }
    },

    async approve (token) {
      let contract = null
      switch (token) {
        case 'CELO':
          contract = this.contracts.goldToken
          break
        case 'cUSD':
          contract = this.contracts.stableToken
          break
      }
      if (contract) {
        const balance = await contract.methods.balanceOf(this.account).call()
        const encodeABI = contract.methods.approve(this.contracts.exchange.options.address, balance).encodeABI()
        this.approve.linkTx = await this.sendTx({
          from: this.account,
          to: contract.options.address,
          data: encodeABI
        })
      }
    },

    async allowance (contract) {
      const allowed = await contract.methods.allowance(this.account, this.contracts.exchange.options.address).call()
      return allowed
    },

    onChangeFrom (event) {
      let temp = 0
      switch (event.target.value) {
        case 'CELO':
          this.exchange.selected = 0
          this.exchange.from.icon = this.exchange.buckets[0].icon
          this.exchange.to.icon = this.exchange.buckets[1].icon
          temp = this.exchange.from.value
          this.exchange.from.value = this.exchange.to.value
          this.exchange.to.value = temp
          break
        case 'cUSD':
          this.exchange.selected = 1
          this.exchange.from.icon = this.exchange.buckets[1].icon
          this.exchange.to.icon = this.exchange.buckets[0].icon
          temp = this.exchange.from.value
          this.exchange.from.value = this.exchange.to.value
          this.exchange.to.value = temp
          break
      }
    },

    async getSellTokenAmount () {
      // getSellTokenAmount(uint256 buyAmount, bool sellGold) external view returns (uint256)
      const buyAmount = this.kit.web3.utils.toWei(this.exchange.from.value)
      const sellGold = this.exchange.selected === 0
      const getSellTokenAmount = await this.contracts.exchange.methods.getSellTokenAmount(buyAmount, sellGold).call()
      this.exchange.to.value = this.kit.web3.utils.fromWei(getSellTokenAmount)
    },

    async getBuyTokenAmount () {
      // getBuyTokenAmount(uint256 sellAmount, bool sellGold) external view returns (uint256)
      const sellAmount = this.kit.web3.utils.toWei(this.exchange.from.value)
      const sellGold = this.exchange.selected === 0
      const getBuyTokenAmount = await this.contracts.exchange.methods.getBuyTokenAmount(sellAmount, sellGold).call()
      this.exchange.to.value = this.kit.web3.utils.fromWei(getBuyTokenAmount)
    },

    async swap () {
      let encodeABI = null
      const contract = this.contracts.exchange
      if (this.exchange.type === 'Sell') {
        const sellAmount = this.kit.web3.utils.toWei(this.exchange.from.value)
        const minBuyAmount = parseInt(sellAmount * 0.9).toString() // TODO: check sol
        const sellGold = this.exchange.selected === 0

        // sell(uint256 sellAmount, uint256 minBuyAmount, bool sellGold)
        // encodeABI = await contract.methods.sell(sellAmount, minBuyAmount, sellGold).encodeABI()

        // exchange(uint256 sellAmount, uint256 minBuyAmount, bool sellGold)
        encodeABI = await contract.methods.exchange(sellAmount, minBuyAmount, sellGold).encodeABI()
      } else {
        const buyAmount = this.kit.web3.utils.toWei(this.exchange.from.value)
        const maxSellAmount = this.kit.web3.utils.toWei(this.exchange.to.value)
        const buyGold = this.exchange.selected === 1

        // buy(uint256 buyAmount, uint256 maxSellAmount, bool buyGold)
        encodeABI = await contract.methods.buy(buyAmount, maxSellAmount, buyGold).encodeABI()
      }
      this.exchange.linkTx = await this.sendTx({
        from: this.account,
        to: contract.options.address,
        data: encodeABI
      })
    },

    async getBuyAndSellBuckets () {
      // getBuyAndSellBuckets(bool sellGold) public view returns (uint256, uint256)
      this.exchange.buckets[0].value = LOADING
      this.exchange.buckets[1].value = LOADING
      const buckets = await this.contracts.exchange.methods.getBuyAndSellBuckets(false).call()
      // (currentGoldBucket, currentStableBucket)
      this.exchange.buckets[0].value = this.kit.web3.utils.fromWei(buckets[0])
      this.exchange.buckets[1].value = this.kit.web3.utils.fromWei(buckets[1])
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
