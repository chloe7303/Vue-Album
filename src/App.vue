<template lang="pug">
  div#app
    b-navbar(toggleable='lg' type='dark' variant='dark')
      b-container
        b-navbar-brand(to='/') 線上相簿
        b-navbar-toggle(target='nav-collapse')
        b-collapse#nav-collapse(is-nav)
          b-navbar-nav.ml-auto
            b-nav-item(v-if="user.id.length === 0" to='/login') 登入
            b-nav-item(v-if="user.id.length === 0" to='/reg') 註冊
            b-nav-item(v-if="user.id.length > 0" to='/album') 我的相簿
            b-nav-item(v-if="user.id.length > 0" @click="logout") 登出
    router-view
</template>

<script>
export default {
  name: 'App',
  computed: {
    user () {
      return this.$store.state.user
    }
  },
  methods: {
    logout () {
      this.axios.delete(process.env.VUE_APP_API + '/users/logout')
        .then(res => {
          if (res.data.success) {
            this.$swal({
              icon: 'success',
              title: '成功',
              text: '登出成功'
            })
            this.$store.commit('logout')

            if (this.$route.path !== '/') {
              this.$router.push('/')
            }
          } else {
            this.$swal({
              icon: 'error',
              title: '錯誤',
              text: res.data.message
            })
          }
        })
        .catch(error => {
          this.$swal({
            icon: 'error',
            title: '錯誤',
            text: error.response.data.message
          })
        })
    },
    heartbeat () {
      this.axios.get(process.env.VUE_APP_API + '/users/heartbeat')
        .then(res => {
          if (this.user.id.length > 0) {
            if (!res.data) {
              this.$swal({
                icon: 'error',
                title: '錯誤',
                text: '登入時效已過'
              })
              this.$store.commit('logout')

              if (this.$router.path !== '/') {
                this.$router.push('/')
              }
            }
          }
        })
        .catch(() => {
          alert('發生錯誤')
          this.$store.commit('logout')

          if (this.$router.path !== '/') {
            this.$$router.push('/')
          }
        })
    }
  },
  mounted () {
    this.heartbeat()
    setInterval(() => {
      this.heartbeat()
    }, 5000)
  }
}
</script>
