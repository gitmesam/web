<template lang="pug">
  nav.navbar.is-fixed-top.is-dark(role="navigation" aria-label="main navigation")
    .navbar-brand(style="margin-left: -.75rem")
      .navbar-item
        | &nbsp;&nbsp;&nbsp;
        img#logo.logo(src="/logo.png" srcset="/logo.svg" @click="home()" alt="logo")
        a.is-size-7.has-text-white(href="https://devternity.com" target="_blank")
          span.is-hidden-mobile &nbsp; by DevTernity
      .navbar-item
        Input(placeholder="Search for videos...") 
      .navbar-burger.burger.has-text-white(data-target="navbarMenu" @click="toggle()" v-bind:class="{ 'is-active': active }")
        span
        span
        span
    .navbar-menu#navbarMenu(style=" margin-right: -.75rem" v-bind:class="{ 'is-active': active }")
      .navbar-start
        router-link.navbar-item(v-if="auth.user && hasSubscriptions" :to='{ name: "search", query: { feed: "true" } }' @click.native="hide()") Subscriptions
        router-link.navbar-item(title="Watched" v-if="watchedCount"  :to='{ name: "search", query: { w: "true" } }' @click.native="hide()") 
          font-awesome-icon(:icon="['far', 'eye']")
          font-awesome-layers(style="position: relative; top: -5px")
            font-awesome-layers-text(:value="watchedCount" transform="shrink-4")
        router-link.navbar-item(title="Favorites" v-if="favoriteCount" :to='{ name: "search", query: { f: "true" } }' @click.native="hide()") 
          font-awesome-icon(:icon="['far', 'star']")
          font-awesome-layers(style="position: relative; top: -5px")
            font-awesome-layers-text(:value="favoriteCount" transform="shrink-4")          
        a.navbar-item(title="Tags" @click="$refs.tags.toggleCollapse()")
          i.fas.fa-hashtag
        ExpandableTags(ref="tags" title="Tags" items="tags" :limit="10" type="tag" attr="tag")
          template(slot-scope="slot") {{slot.item.tag | capitalizeIfNeeded}}
        a.navbar-item(title="Speakers" @click="$refs.speakers.toggleCollapse()")
          i.far.fa-user-circle
        ExpandableTags(ref="speakers" title="Speakers" items="speakers" :limit="10" type="speaker" attr="twitter")
          template(slot-scope="slot") {{slot.item.name | capitalizeIfNeeded}}
        a.navbar-item(title="Channels" @click="$refs.channels.toggleCollapse()")
          i.fab.fa-youtube
        ExpandableTags(ref="channels" title="Channels" items="channels" :limit="10" type="channel" attr="title")
          template(slot-scope="slot") {{slot.item.title | truncate(25) | capitalizeIfNeeded}}
        slot
      .navbar-end
        a.navbar-item.is-hoverable(v-if="auth.user")
          .face.is-hidden-touch(:style="'background-image: url(' + auth.user.photoUrl + ')'")
          font-awesome-icon.is-hidden-touch(icon="ellipsis-v")
          span.is-hidden-desktop {{auth.user.name}}
          .navbar-dropdown.is-right.is-boxed
            NightMode
            a.navbar-item(@click="signOut()") Logout

        a.navbar-item.is-hoverable(v-else) 
          font-awesome-icon(:icon="['fa', 'user']")
          | &nbsp;Log in
          .navbar-dropdown.is-right.is-boxed
            a.navbar-item(@click="signIn('github'); hide()") via Github
            a.navbar-item(@click="signIn('twitter'); hide()") via Twitter
            a.navbar-item(@click="signIn('google'); hide()") via Google
        | &nbsp;&nbsp;&nbsp;
</template>
<style lang="scss" scoped>
header {
  
  .logo {
    cursor: pointer;
  }

  .navbar-menu .navbar-item {
    font-size: 12px;
  }

  .face {
    height: 30px;
    width: 30px;
    background-size: cover;
    background-repeat: no-repeat;
    overflow: hidden;
    border-radius: 50%;
    border: 1px solid white;
    margin-right: 0.5em;
  }

  @media only screen and (max-width: 768px) {
    .logo {
      width: 45px;
    }
  }

}
</style>
<script>
  import Input from './Input.vue'
  import NightMode from './NightMode.vue'
  import ExpandableTags from './ExpandableTags.vue'

  import { mapState, mapActions, mapGetters } from 'vuex'


  export default {
    computed: {
      ...mapState([ 'auth' ]),
      ...mapGetters('videos', [ 'watchedCount', 'favoriteCount', 'hasSubscriptions' ])
    },
    data: function() {
      return {
        active: false
      }
    },
    methods: {
      // double check if toggle() is not used in children components
      toggle() {
        this.active = !this.active
      },
      hide() {
        this.active = false
      },
      home() {
        window.location = "/"
      },
      ...mapActions('auth', [ 'signOut', 'signIn']),
    },
    components: { Input, NightMode, ExpandableTags }  
  }
</script>