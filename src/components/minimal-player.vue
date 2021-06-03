<style lang="scss">
  .ar-player {
    width: 200px;
    height: 45px;
    padding: 0 10px;
    margin: 0 auto;
    line-height: 45px;
    display: flex;
    justify-content: space-between;
    border-bottom: 1px solid #E8E8E8;
    position: relative;

 
    &__play {
      position: absolute;
      top: 10px;
      right: 90px;
      background-color: #FFFFFF;
      box-shadow: 0 2px 11px 11px rgba(0,0,0,0.07);

      &--active {
        fill: white !important;
        background-color: #05CBCD !important;

        &:not(.disabled):hover{
          fill: #505050 !important;
        }
      }
    }
  }

  div.disabled {
    color: grey;
    border-color: white;
    // pointer-events: none;
    opacity: .6;
    cursor: not-allowed !important;
    user-select: none;
    &:hover {
      cursor: not-allowed !important;
    }
  }

  @media (min-device-width: 320px) and (max-device-width: 700px) {
    .ar-player {
      width: 85vw;
    }
  }
  @import '../scss/icons';
</style>

<template>
<div>
      <icon-button
        id="play"
        class="ar-icon ar-icon__sm ar-player__play"
        :name="playBtnIcon"
        :class="{'ar-player__play--active': isPlaying, 'disabled': disablePlayButton}"
        @click.native="playback"/>

    <audio :id="playerUniqId" :src="audioSource"></audio>
  </div>
</template>

<script>
  import IconButton    from './icon-button'
  import LineControl   from './line-control'
  import VolumeControl from './volume-control'
  import { convertTimeMMSS } from '@/lib/utils'

  export default {
    props: {
      src      : { type: String },
      record   : { type: Object },
      filename : { type: String },
      showPlayerProgress : {type: Boolean, default: true}
    },
    data () {
      return {
        isPlaying  : false,
        duration   : convertTimeMMSS(0),
        playedTime : convertTimeMMSS(0),
        progress   : 0,
        disablePlayButton : true,
      }
    },
    components: {
      IconButton,
      LineControl,
      VolumeControl
    },
    mounted: function() {
      this.player = document.getElementById(this.playerUniqId)

      this.player.addEventListener('ended', () => {
        this.isPlaying = false
      })

      this.player.addEventListener('loadeddata', (ev) => {
        this._resetProgress()
        this.duration = convertTimeMMSS(this.player.duration)
      })

      this.player.addEventListener('timeupdate', this._onTimeUpdate)

      this.$eventBus.$on('remove-record', () => {
        this._resetProgress()
      })
    },
    computed: {
      audioSource () {
        const url = this.src || this.record.url
        if (url) {
          this.disablePlayButton = false
          return url
        } else {
          this.disablePlayButton = true
          this._resetProgress()
        }
      },
      playBtnIcon () {
        return this.isPlaying ? 'pause' : 'play'
      },
      playerUniqId () {
        return `audio-player${this._uid}`
      }
    },
    methods: {
      playback () {
        if (!this.audioSource) {
          return
        }

        if (this.isPlaying) {
          this.player.pause()
        } else {
          setTimeout(() => { this.player.play() }, 0)
        }

        this.isPlaying = !this.isPlaying
      },
      _resetProgress () {
        if (this.isPlaying) {
          this.player.pause()
        }

        this.duration   = convertTimeMMSS(0)
        this.playedTime = convertTimeMMSS(0)
        this.progress   = 0
        this.isPlaying  = false
      },
      _onTimeUpdate () {
        this.playedTime = convertTimeMMSS(this.player.currentTime)
        this.progress = (this.player.currentTime / this.player.duration) * 100
      },
      _onUpdateProgress (pos) {
        if (pos) {
          this.player.currentTime = pos * this.player.duration
        }
      },
      _onChangeVolume (val) {
        if (val) {
          this.player.volume = val
        }
      }
    }
  }
</script>
