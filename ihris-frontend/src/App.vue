<template>
  <v-app id="top">
    <the-header app :header="header" v-on:loggedin="updateConfig" v-on:loggedout="updateConfig" />
    <the-navigation app :nav="nav" v-on:loggedin="updateConfig" />

    <v-content>
      <v-snackbar
        app
        class="mt-12"
        v-model="$store.state.message.active"
        :color="$store.state.message.type"
        :timeout="$store.state.message.timeout"
        top
        multi-line
        >
        {{ $store.state.message.text }}
          <v-btn icon dark @click="$store.commit('closeMessage')">
            <v-icon>mdi-close</v-icon>
          </v-btn>
      </v-snackbar>

      <v-dialog
        v-model="$store.state.progress.enabled"
        persistent
        width="300"
      >
        <v-card
          color="primary"
          dark
        >
          <v-card-text>
            {{$store.state.progress.title}}
            <v-progress-linear
              indeterminate
              color="white"
              class="mb-0"
            ></v-progress-linear>
          </v-card-text>
        </v-card>
      </v-dialog>
      <router-view :key="$route.fullPath"></router-view>
      <router-view v-if="$store.state.user.loggedin" name="homeNav" :nav="nav" :key="$route.fullPath"></router-view>
    </v-content>

    <the-footer :footer="footer" />
  </v-app>
</template>

<style>
.ihris-sections-menu {
  position: sticky;
  top: 0;
  z-index: 2;
}
.ihris-sections-menu::before {
  display: block;
  content: " ";
  margin-top: -70px;
  height: 70px;
  visibility: hidden;
  pointer-events: none;
}
.ihris-section::before {
  display: block;
  content: " ";
  margin-top: -70px;
  height: 70px;
  visibility: hidden;
  pointer-events: none;
}
</style>

<script>
import { eventBus } from "@/main";
import TheHeader from "./components/layout/the-header"
import TheNavigation from "./components/layout/the-navigation"
import TheFooter from "./components/layout/the-footer"

export default {
  name: "App",
  data: () => ({
    header: {
      title: false,
      site: null,
      logo: "iHRIS5Logo.png",
      auths: [],
    },
    footer: {
      links: []
    },
    nav: {
      active: null,
      menu: {},
      auths: []
    }
  }),
  components: {
    TheHeader,
    TheNavigation,
    TheFooter
  },
  methods: {
    updateConfig: function() {
      // make sure we're user has been created in session (logged in or not)
      fetch("/auth").then(()=> {
        fetch("/config/site").then(response => {
          response.json().then(data => {
            this.$store.state.version = data.version
            if (data.auth && data.auth.signup) {
              this.$store.state.signup = data.auth.signup
            }
            if(data.hasOwnProperty("footer")) {
              if (data.footer.hasOwnProperty("links")) {
                this.footer.links = []
                for(let id of Object.keys(data.footer.links)) {
                  data.footer.links[id].id = id
                  this.footer.links.push(data.footer.links[id])
                }
              }
            }
            if(data.hasOwnProperty("login")) {
              if (data.login.hasOwnProperty("links")) {
                this.$store.state.login.links = []
                for(let id of Object.keys(data.login.links)) {
                  data.login.links[id].id = id
                  this.$store.state.login.links.push(data.login.links[id])
                }
              }
            }
            if (data.hasOwnProperty("security") && data.security.hasOwnProperty("disabled")) {
              this.$store.commit('securityOff', data.security.disabled)
            }
            if (data.hasOwnProperty("title")) this.header.title = data.title
            if (data.hasOwnProperty("site")) this.header.site = data.site
            if (data.hasOwnProperty("logo")) this.header.logo = data.logo
            if (data.hasOwnProperty("auth")) {
              this.header.auths = []
              this.nav.auths = []
              for(let id of Object.keys(data.auth)) {
                data.auth[id].id = id
                this.header.auths.push(data.auth[id])
                this.nav.auths.push(data.auth[id])
              }
            }
            if (data.hasOwnProperty("user")) {
              if ( data.user.loggedin ) {
                let user = {
                  name: data.user.name,
                  role: data.user.role,
                  obj: data.user.obj
                };
                this.$store.commit('login', user )
              } else {
                this.$store.commit('logout')
              }
            }
            if (data.hasOwnProperty("nav")) {
              if (data.nav.hasOwnProperty("active")) this.nav.active = data.nav.active
              if (data.nav.hasOwnProperty("menu")) this.nav.menu = data.nav.menu
              if (data.nav.hasOwnProperty("home")) {
                this.nav.home = data.nav.home
              }
            }
            if (!this.$store.state.user.obj.resource || this.$store.state.user.obj.resource.id === 'ihris-user-loggedout') {
              if (this.$route.path !== '/') {
                this.$router.push("/")
              }
            }
            if(this.$store.state.user.loggedin && data.pages && data.pages.home && data.pages.home.url) {
              this.$router.push(data.pages.home.url)
            }
          })
        })
      })
    }
  },
  created: function() {
    if(!localStorage.getItem('activeLang')){
      localStorage.setItem('activeLang', "English")
      localStorage.setItem('activeFlag', "en")
      localStorage.setItem('activeCode', "en")
    }
    this.updateConfig()
    eventBus.$on("updateconfig", () => {
      this.updateConfig()
    })
  }
}
</script>
