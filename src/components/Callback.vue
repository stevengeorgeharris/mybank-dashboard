<template>
  <h1>{{ msg }}</h1>
</template>

<script>
import Axios from 'axios'
import Settings from '../config/settings'

function getURLParameter (name) {
  return decodeURIComponent((new RegExp('[?|&]' + name + '=' + '([^&;]+?)(&|#|;|$)').exec(location.search) || [null, ''])[1].replace(/\+/g, '%20')) || null
}

export default {
  name: 'callback',

  created () {
    let authCode = getURLParameter('code')

    const config = {
      headers: {'Content-Type': 'application/x-www-form-urlencoded'}
    }

    let options = {
      'grant_type': 'authorization_code',
      'client_id': Settings.client_id,
      'client_secret': Settings.client_secret,
      'redirect_uri': 'http://localhost:8080/callback',
      'code': authCode
    }

    const searchParams = Object.keys(options).map((key) => {
      return encodeURIComponent(key) + '=' + encodeURIComponent(options[key])
    }).join('&')

    Axios.post('https://api.monzo.com/oauth2/token', searchParams, config)
      .then(res => {
        console.log(res.data)
        localStorage.setItem('auth_code', res.data.access_token)
        localStorage.setItem('user_id', res.data.user_id)
        localStorage.setItem('refresh_token', res.data.refresh_token)
        localStorage.setItem('auth_expire', res.data.expires_in)
      })
      .then(() => {
        window.location = '/'
      })
      .catch(err => {
        console.log(err)
      })
  },
  data () {
    return {
      msg: 'Thinking...'
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
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
