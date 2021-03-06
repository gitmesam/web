<template lang="pug">
  section.section.body
    .container
      .columns
        .column.is-9
          .notification.is-primary(v-if="completed") This brown-bag has completed {{completedOn}}. 
            a(:href="'/video/' + brownbag.videoId") Watch recording
            | , discover other brown-bags or 
            a create your own
            | .
          .notification.is-primary(v-if="cannotAutoplay") Autoplay does not work in your browser. Press 
            strong
              a(@click="play()") play
            |  to watch brown-bag session.

          .notification.is-primary(v-if="soon")
            p coming soon {{startsOn}}

          h1.title.is-uppercase.is-size-5.is-marginless {{this.brownbag.title}}
          span(v-if="profile").is-size-7 {{profile.name | noemoji}}
          span  &nbsp;·&nbsp; 
          a.is-size-7.is-lowercased(:href="'https://twitter.com/' + brownbag.speaker" rel="nofollow" target="_blank")
            i.fab.fa-twitter
            |  {{brownbag.speaker}}
          br
          br
          .videoWrapper(v-if="live")
            .top.heading
              font-awesome-icon.has-text-danger(:icon="['fas', 'circle']")
              span  35 watching
            video-player(ref="player" :options="videoOptions" :starts="brownbag.datetime" :duration="duration" v-on:autoplayPrevented="autoplayPrevented")
          .image.is-16by9(v-else :style="'background-image: url(//img.youtube.com/vi/' + brownbag.videoId + '/hqdefault.jpg)'")
          Hearts(ref="hts" :room="id")
        .column.is-3
          Chat(:room="id" v-on:heart-pressed="loveInTheAir()")
          
    br
    br
    .columns
      //- .column.is-4
    .columns

</template>
<style lang="scss">
  .is-16by9 {
    background-size: cover;
  }
  .videoWrapper {
    position: relative;
      .top {
        background-color: black;
        opacity: 0.7;
        padding: 5px;
        position: absolute;
        left: 5px;
        top: 5px;
        z-index: 99;
      }
  }  

  
</style>
<script>
  import Hearts from './Hearts.vue'
  import SpeakerStats from './SpeakerStats.vue'
  import VideoPlayer from './VideoPlayer.vue'
  import Chat from './Chat.vue'
  import axios from 'axios'
  import * as dayjs from 'dayjs'
  import utc from 'dayjs/plugin/utc'
  import isBetween from 'dayjs/plugin/isBetween'
  import relativeTime from 'dayjs/plugin/relativeTime'
  dayjs.extend(utc)
  dayjs.extend(isBetween)
  dayjs.extend(relativeTime)

  export default {
    props: {
      id: { type: String, required: true },
    },
    asyncComputed: {
      profile() {
        if (brownbag)
          return 
            axios.get(`https://dossier.dev.tube/twt/` + brownbag.speaker).then(response => response.data).catch(err => console.err(err))
      }
    },    
    created() {
      this.brownbag = window.brownbag
    },
    computed: {
      completed() {
        return dayjs().isAfter(this.endsAt())
      },
      live() {
        return dayjs().isBetween(this.brownbag.datetime, this.endsAt(), '[]')
      },
      soon() {
        return dayjs().isBefore(dayjs(this.brownbag.datetime))
      },
      startsOn() {
        return dayjs(this.brownbag.datetime).fromNow()
      },
      completedOn() {
        return dayjs(this.endsAt()).fromNow()
      }      
    },
    data() {
      return {
          cannotAutoplay: false,
          duration: 3319,
          videoOptions: {
                aspectRatio: "16:9",
                autoplay: true,
                controls: false,
                preload: 'auto',
                replay: false,
                loop: false,
                sources: [
                    {
                        src: brownbag.source,
                        type: "application/x-mpegURL"
                    }
                ]
            }
      }
    },
    methods: {
      loveInTheAir() {
          this.$refs.hts.send()
      },
      endsAt() {
        return dayjs(this.brownbag.datetime).add(this.duration, 'second')
      },
      play() {
        this.$refs.player.play()
        this.cannotAutoplay = false
      },
      autoplayPrevented() {
        this.cannotAutoplay = true
      }
    },
    components: { VideoPlayer, Chat, Hearts }
  }
</script>