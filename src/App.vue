<template>
  <div id="app">
      <div class="container">
            <div class="col-12 col-md-6 offset-md-3 mt-4">
                <div class="card">
                    <div class="card-header">
                       <h6>Welcome to Bank2U</h6>
                    </div>
                    <div class="card-body">
                        <button id="connectButton"
                            v-if="!currentAccount"
                            v-on:click="connectWallet"
                            :disabled="connecting"
                            class="btn btn-success">{{ (!connecting) ? 'Connect Metamask' : 'Connecting ... '}}</button>

                        <div v-if="currentAccount">
                        <div class="d-flex flex-column justify-content-center align-items-center mb-4">
                          <div>
                            <b>My Balance in Bank</b>
                          </div>
                          <div class="d-flex flex-row">
                            <h2 class="m-0">{{ currentBalance }}</h2>
                            <small class="text-muted align-self-end mb-2 ml-1">ETH</small>
                          </div>
                        </div>

                          <div class="form-group">
                            <label for="withdraw">Withdraw (Only Signer can withdraw)</label>
                            <div class="row">
                              <div class="input-group col-10 mb-2">
                                <input type="string" id="withdrawaddress" 
                                :disabled="loading"
                                v-model="form.withdrawalAddress" class="form-control"/>
                              </div>
                              <div class="input-group col-10">
                                <input type="number" id="withdraw" 
                                :disabled="loading"
                                v-model="form.withdrawalAmount" class="form-control"/>
                                  <div class="input-group-append">
                                    <span class="input-group-text" id="basic-addon2">ETH</span>
                                  </div>
                              </div>
                              <button class="btn btn-danger col-2 btn-sm"
                               :disabled="loading"
                               v-on:click.prevent="withdraw">Withdraw</button>
                            </div>
                          </div><!--/.form-group-->

                        <label for="withdraw">Subscribe!</label>
                          <div class="form-group">
                            <div class="card-body">
                              <button class="btn btn-success btn-bg mb-2 ml-1"
                                :disabled="loading"
                                      v-on:click.prevent="subscribe(0.05)">1 Month/0.05 ETH</button>
                              <button class="btn btn-success btn-bg mb-2 ml-1"
                                :disabled="loading"
                                      v-on:click.prevent="subscribe(0.1)">3 Month/0.1 ETH</button>
                              <button class="btn btn-success btn-bg mb-2 ml-1"
                                :disabled="loading"
                                      v-on:click.prevent="subscribe(0.15)">9 Month/0.15 ETH</button>
                              <button class="btn btn-success btn-bg mb-2 ml-1"
                                :disabled="loading"
                                      v-on:click.prevent="subscribe(0.25)">1 Year/0.25 ETH</button>
                            </div>
                          </div>  
                        </div><!--/.flex-col-->
                    </div><!--/.card-body-->
                </div><!--/.card-->

                <div class="card">

                </div>

            </div>
      </div>
  </div>
</template>

<script>
import {ethers} from "ethers";
import ContractAbi from '../NewContract/Contract.json';
export default {
  name: 'App',
  components: {
  },
  data () {
    return {
      contract: null,
      loading: false,
      connecting: false,
      contractInitialised: false,
      currentAccount: null,
      currentBalance: 0,
      form: {
        depositAmount: 0,
        withdrawalAddress: "",
        withdrawalAmount: 0,
      }
    }
  },
  mounted () {
    // metamask
    const { ethereum } = window

    // initialise contract
    if (ethereum) {
      const provider = new ethers.providers.Web3Provider(ethereum)
      const signer = provider.getSigner()
      const contract = new ethers.Contract(
        '0x36E7f1fE02DDD8A77a5aE99d1F408Bf5a97238Bb', // change this when deploy new contract
        ContractAbi.abi,
        signer
      )
      this.contract = Object.freeze(contract)
      console.log('Contract initialised')
    } else {
      this.contractInitialised = false
    }
  },
  methods: {
    // connect metamask wallet
    async connectWallet () {
      try {
        const { ethereum } = window

        // check if metamask is found
        if (!ethereum) {
          console.log('Metamask not detected')
          return
        }

        // get accounts
        this.connecting = true
        const accounts = await ethereum.request({method: 'eth_requestAccounts'})

        this.connecting = false
        console.log('Found account', accounts[0])
        this.currentAccount = accounts[0] // set first account to currentAccount

        // check balance of account
        this.checkBalance()

      } catch (error) {
        this.connecting = false
        console.log('Error connecting to metamask', error)
      }
    },

    // subscribe
    async subscribe (subAmount) {
      try {
        this.loading = true
        const options = {
          from: this.currentAccount,
          value: ethers.utils.parseEther(parseFloat(subAmount).toString())
        }

        // call contract deposit method
        let txn = await this.contract.deposit(options)
        let txnResults = await txn.wait() // await for transaction to be mined
        console.log(txnResults)

        alert(`Successfully subscribed ${subAmount} ETH to bank`)
        this.checkBalance() // initiate check balance

        this.loading = false
        this.form.depositAmount = 0

      } catch (e) {
        console.log(e)
        this.loading = false
        alert('Failed to subscribe')
      }
    },

    // withdraw ether from bank
    async withdraw () {
      try {
        /*
        const options = {
          from: this.currentAccount,
        }
        */
        this.loading = true
        console.log(this.withdrawalAddress)

        // call contract withdraw method
        let txn = await this.contract.withdraw(
          this.form.withdrawalAddress,
          ethers.utils.parseEther(parseFloat(this.form.withdrawalAmount).toString())
        )
        let txnResults = await txn.wait() // await for transaction to be mined
        console.log(txnResults)

        alert(`Successfully withdrew ${this.form.withdrawalAmount} ETH from bank`)
        this.checkBalance() // initiate check balance

        this.loading = false
        this.form.withdrawalAmount = 0
      } catch (e) {
        console.log(e)
        this.loading = false
        alert('Failed to withdraw')
      }
    },

    // get current balance of account
    async checkBalance () {
        this.loading = true

        // call contract balance method
        let balance = await this.contract.getContractBalance(this.currentAccount)

        // convert wei to ether
        this.currentBalance = ethers.utils.formatEther(balance) // 1000000000000 = 1 ETH
        this.loading = false
    },
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&display=swap');

html, body, p, span, div {
    font-family: 'Roboto', sans-serif;
}

body {
    background-color: #242424;
}
</style>
