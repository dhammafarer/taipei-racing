<template>
  <div id="root">

    <header>
      <header-bar></header-bar>
    </header>

    <main>
      <div class="content">
        <notifications></notifications>
        <router-view v-if="!loading" transition-mode="out-in" transition="fade"></router-view>
      </div>
    </main>

    <footer>
      <footer-bar></footer-bar>
    </footer>

  </div>
</template>

<script>
  import io from 'socket.io-client'
  import auth from '../auth'
  import store from '../vuex/store'
  import HeaderBar from './HeaderBar.vue'
  import FooterBar from './FooterBar.vue'
  import Notifications from './Notifications.vue'
  import { fetchAllRaces, socketRaceUpdated } from '../vuex/races/actions'
  import { fetchAllDrivers } from '../vuex/drivers/actions'
  import { addNotification, closeNotification } from '../vuex/notifications/actions'

  export default {
    store: store,
    components: {
      HeaderBar,
      FooterBar,
      Notifications
    },
    vuex: {
      actions: {
        fetchAllRaces,
        socketRaceUpdated,
        fetchAllDrivers,
        addNotification,
        closeNotification
      }
    },
    data () {
      return {
        loading: false
      }
    },
    created () {
      let notification = { title: 'Loading', body: '', type: 'loading' }
      this.addNotification(notification)
      this.loading = true
      Promise.all([this.fetchAllRaces(), this.fetchAllDrivers()])
        .then(() => {
          this.closeNotification(notification)
          this.loading = false
        })
    },
    ready () {
      let socket = io()
      socket.on('races-channel:App\\Events\\RaceUpdated', (data) => {
        this.socketRaceUpdated(data)
      })

      auth.checkAuth()
    }
  }
</script>
