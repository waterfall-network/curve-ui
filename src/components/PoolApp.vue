<template>
  <div id="app">
     <div class="top-menu-bar">
      <label for="hamburger" class='border-menu'></label>
      <input type="checkbox" id="hamburger"/>

      <div class='poolsdropdown'>
        <button class='simplebutton' :class="{'loading line': !initializedContracts && !['Stats', 'FAQ', 'Donate'].includes($route.name)}">[{{poolMenu[currentPool]}}]</button>
        <div class='dropdown'>
            <button class='simplebutton' @click = 'changeWallets'>Change wallet</button>
            <button id='changeAccounts' class='simplebutton' 
              v-show="['ledger', 'trezor'].includes(walletName)" 
              @click = 'changeAccounts'>Change accounts</button>
        </div>
      </div>

      <router-link to='/'>Root</router-link>
      <router-link to='/combinedstats' class='showmobile'>All stats</router-link>
      <router-link :to="'/'+currentPool" v-show="currentPool !='susd'">Buy and sell</router-link>
      <router-link :to="'/' + currentPool + '/deposit'" v-show="currentPool !='susd'">Deposit</router-link>
      <router-link :to="'/' + currentPool + '/withdraw'">Withdraw</router-link>
<!--      <router-link :to="'/' + currentPool + '/withdraw_old'" v-show="currentPool == 'compound' && oldBalance > 0">Withdraw old</router-link>-->
<!--      <router-link to="/susd/withdraw" v-show="currentPool == 'susdv2' && oldBalance > 0">Withdraw old</router-link>-->
<!--      <router-link :to="'/' + currentPool + '/stats'" v-show="currentPool !='susd'">Stats</router-link>-->
<!--      <router-link :to="'/' + currentPool + '/profit'" v-show="currentPool !='susd'">Profit</router-link>-->
<!--      <router-link :to="'/curvepay/' + currentPool">Pay</router-link>-->
<!--      <div class='poolsdropdown right'>-->
<!--        <span>?</span>-->
<!--        <div class='dropdown'>-->
<!--          <a :href="'https://explorer.waterfall.network/address/' + this.poolAddress" rel='noopener noreferrer'>Pool contract</a>-->
<!--          <a :href="'https://explorer.waterfall.network/address/' + this.tokenAddress" rel='noopener noreferrer'>Token contract</a>-->
<!--          <p>____________</p>-->
<!--          <router-link to="/audits">Audits</router-link>-->
<!--          <router-link to="/events">Events</router-link>-->
<!--          <router-link :to="'/' + currentPool + '/risks'">Risks</router-link>-->
<!--          <router-link to="/bugbounty">Bug Bounty</router-link>-->
<!--          <router-link :to="'/' + currentPool + '/faq'">FAQ</router-link>-->
<!--          <router-link to="/integrations">Integrations</router-link>-->
<!--          <router-link :to="'/' + currentPool + '/donate'">Donate</router-link>-->
<!--          <a href='https://guides.curve.fi' rel='noopener noreferrer'>Guides</a>-->
<!--          <p>____________</p>-->
<!--          <a :href="'https://github.com/curvefi/curve-contract/tree/pool_'+gitBranches[currentPool]" rel='noopener noreferrer'>git@</a>-->
<!--          <a href="https://github.com/pengiundev/curve-vue" rel='noopener noreferrer'>git@UI</a>-->
<!--        </div>-->
<!--      </div>-->
<!--      <router-link to="/audits" class='showmobile'>Audits</router-link>-->
<!--      <router-link to="/events" class='showmobile'>Events</router-link>-->
<!--      <router-link :to="'/' + currentPool + '/faq'" class='showmobile'>FAQ</router-link>-->
<!--      <router-link to="/integrations" class='showmobile'>Integrations</router-link>-->
<!--      <router-link to="/bugbounty" class='showmobile'>Bug Bounty</router-link>-->
<!--      <router-link :to="'/' + currentPool + '/donate'" class='showmobile'>Donate</router-link>-->
<!--      <a href='https://guides.curve.fi' rel='noopener noreferrer' class='showmobile'>Guides</a>-->
<!--      <a :href="'https://github.com/curvefi/curve-contract/tree/pool_'+gitBranches[currentPool]" class='showmobile' rel='noopener noreferrer'>git@</a>-->
<!--      <a href="https://github.com/pengiundev/curve-vue" class='showmobile' rel='noopener noreferrer'>git@UI</a>-->
      <button class='simplebutton showmobile' @click = 'changeWallets'>Change wallet</button>
    </div>
    <div id="screen">
        <div :class="{'blue window': true, [$route.name]: true}">
            <h1><img :src="logoSrc" alt="🌀 Curve"></h1>
        </div>
        <div class="error window half-width info" id="error-window" v-show='error'>
          {{error}}
        </div>
        <div class='info-message gentle-message window half-width gentle-message' v-show='hasConnectedWallet'>
          You haven't connected a wallet. <button @click='changeWallets'>Connect wallet</button>
        </div>
        <router-view/>
    </div>
    <balances-info
    :class = '{[$route.name]: true}'
    :bal_info = 'bal_info'
    :total = 'balTotal'
    :l_info = 'l_info'
    :totalShare = 'totalShare'
    :staked_info = 'staked_info'
    :totalStake = 'totalStake'
    :fee = 'fee'
    :admin_fee = 'admin_fee'
    :currencies = 'currencies'
    v-if="!['Stats', 'FAQ', 'Donate', 'Root', 'CombinedStats'].includes($route.name)"/>

    <TotalBalances></TotalBalances>

    <footer>
      <a href="https://explorer.waterfall.network">Waterfall network explorer</a>
    </footer>
  </div>
</template>

<script>
  import BalancesInfo from '../components/BalancesInfo.vue'
  import { getters, contract as currentContract, changeContract, poolMenu } from '../contract'
  import init, { onboard } from '../init'
  import allabis from '../allabis'
  import TotalBalances from "./root/TotalBalances";

  export default {
    data: () => ({
      gitBranches: {
        compound: 'compound',
        usdt: 'usdt',
        iearn: 'y',
        y: 'y',
        busd: 'busd',
        susd: 'susd_vulnerable',
        susdv2: 'susd_plain',
        pax: 'pax',
        tbtc: 'tbtc',
        ren: 'ren',
        sbtc: 'renbtc_sbtc',
      }
    }),
    components: {
      TotalBalances,
      BalancesInfo,
    },
    computed: {
      allGetters() {
        return getters;
      },
      ...getters,
      poolMenu() {
        return poolMenu;
      },
      poolAddress() {
        return allabis[this.currentPool].swap_address
      },
      tokenAddress() {
        return allabis[this.currentPool].token_address
      },
      publicPath() {
        return process.env.BASE_URL
      },
      logoSrc() {
        if(!currentContract.swapbtc) return this.publicPath + 'logo_optimized.png'
      },
      hasConnectedWallet() {
        return this.default_account == '0x0000000000000000000000000000000000000000'
                && !['Donate', 'StatsDaily', 'Audits', 'Stats', 'Contracts', 'FAQ', 'RootFAQ'].includes(this.$route.name)
      },
    },
    methods: {
      changePools(pool) {
        changeContract(pool)
      },
      async changeWallets() {
        currentContract.default_account = ''
        onboard.walletReset()
        localStorage.removeItem('selectedWallet')
        currentContract.totalShare = 0
        let userSelectedWallet = await onboard.walletSelect();
        await onboard.walletCheck();
      },
      async changeAccounts() {
        return onboard.accountSelect();
      },
    },
  }
</script>

<style>
  #changeAccounts {
    margin-top: 0.3em;
  }
  a.showmobile {
    display: none;
  }
  @media only screen and (min-device-width : 320px) and (max-device-width : 730px) {
    #hamburger:checked ~ a.showmobile {
      display: block;
    }
    .blue.window.half-width, .info-message.window.half-width {
      width: 90%;
    }
  }
  h1 > img {
    height: 52.125px;
  }
</style>