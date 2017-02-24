<template>
  <div class="content">
    <button v-if="!loggedIn" @click="authenticate">Sign in</button>
    <div v-else-if="user.length > 0" class="welcome">
      Hello, {{ user }}
    </div>
    <ul class="transactions">
      <li v-for="item in transactions">
        {{ item.description }} <b>{{ currency(item.amount) }}</b>
      </li>
    </ul>
  </div>
</template>

<script>
import Axios from 'axios'
import Settings from '../config/settings'

let authOptions = {
  client_id: Settings.client_id,
  redirect_uri: 'http://localhost:8080/callback',
  response_type: 'code'
}

const config = {
  headers: {
    'Authorization': 'Bearer ' + localStorage.getItem('auth_code')
  }
}

export default {
  name: 'hello',
  data () {
    return {
      user: '',
      accountId: '',
      balance: null,
      transactions: [],
      loggedIn: localStorage.getItem('user_id')
    }
  },
  created () {
    if (this.loggedIn) {
      this.getUser()
    }
  },
  methods: {
    currency (amount) {
      return amount > 0 ? '+' + amount / 100 : amount / 100
    },
    authenticate () {
      window.location = 'https://auth.getmondo.co.uk/?' + 'client_id=' + authOptions.client_id + '&redirect_uri=' + authOptions.redirect_uri + '&response_type=' + authOptions.response_type
    },
    getUser () {
      Axios.get('https://api.monzo.com/accounts', config)
        .then(res => {
          this.user = res.data.accounts[0].description
          this.accountId = res.data.accounts[0].id
          this.getTransactions()
        })
        .catch(err => {
          if (err.response.status === 401) {
            this.loggedIn = false
          }
        })
    },
    getTransactions () {
      Axios.get('https://api.monzo.com/transactions?account_id=' + this.accountId, config)
        .then(res => {
          this.transactions = res.data.transactions
        })
        .catch(err => {
          console.log(err)
        })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: block;
  margin: 1em;
}

a {
  color: #42b983;
}
</style>
