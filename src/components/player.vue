<!--
* Vue-Video-Player
* Author: surmon@foxmail.com
* Github: https://github.com/surmon-china/vue-video-player
* Adapted from Videojs (https://github.com/videojs/video.js)
-->

<template>
  <div class="video-player" v-if="show">
    <video class="video-js" ref="video"></video>
  </div>
</template>

<script>
import videojs from 'video.js'

const DEFAULT_EVENTS = [
  'loadeddata',
  'canplay',
  'canplaythrough',
  'play',
  'pause',
  'waiting',
  'playing',
  'ended',
  'error'
]

export default {
  name: 'Player',

  props: {
    playsinline: {
      type: Boolean,
      default: false
    },
    customEventName: {
      type: String,
      default: 'statechanged'
    },
    options: {
      type: Object,
      required: true,
      default: () => ({
        width: 1000,
        height: 400,
        // autoplay: false,
        controls: true,
        // preload: 'auto',
        // fluid: false,
        // muted: false,
        controlBar: {
          remainingTimeDisplay: false,
          playToggle: {},
          progressControl: {},
          fullscreenToggle: {},
          volumeMenuButton: {
            inline: false,
            vertical: true
          }
        },
        techOrder: ['html5'],
        plugins: {},

        sources: [{
          type: 'video/mp4',
          // mp4
          src: 'http://vjs.zencdn.net/v/oceans.mp4'
        }],
        poster: 'https://surmon-china.github.io/vue-quill-editor/static/images/surmon-1.jpg'
      })
    },
    events: {
      type: Array,
      default: () => []
    }
  },

  data () {
    return {
      player: null,
      show: true
    }
  },

  watch: {
    options: {
      deep: true,
      handler (options) {
        this.dispose(() => {
          if (options && options.sources && options.sources.length) {
            this.initialize()
          }
        })
      }
    }
  },

  mounted () {
    if (!this.player) this.initialize()
  },

  beforeDestroy () {
    if (this.player) this.dispose()
  },

  methods: {
    initialize () {
      const videoOptions = {
        ...this.globalOptions,
        ...this.options
      }

      // ios fullscreen
      if (this.playsinline) {
        this.$refs.video.setAttribute('playsinline', this.playsinline)
        this.$refs.video.setAttribute('webkit-playsinline', this.playsinline)
        this.$refs.video.setAttribute('x5-playsinline', this.playsinline)
        this.$refs.video.setAttribute('x5-video-player-type', 'h5')
        this.$refs.video.setAttribute('x5-video-player-fullscreen', false)
      }

      const emitPlayerState = (event, value) => {
        if (event) this.$emit(event, this.player)
        if (value) this.$emit(this.customEventName, {[event]: value})
      }

      // avoid error "VIDEOJS: ERROR: Unable to find plugin: __ob__"
      if (videoOptions.plugins) {
        delete videoOptions.plugins.__ob__
      }

      // player
      const self = this
      this.player = videojs(this.$refs.video, videoOptions, function () {
        const events = DEFAULT_EVENTS.concat(self.events)

        const eventSet = new Set()
        for (const event of events) {
          if (typeof event !== 'string') continue
          if (eventSet.has(event)) continue

          eventSet.add(event)
          const e = event
          this.on(e, () => {
            emitPlayerState(e, true)
          })
        }

        this.on('timeupdate', function () {
          emitPlayerState('timeupdate', this.currentTime())
        })

        self.$emit('ready', this)
      })
    },

    dispose (callback) {
      if (!this.player) return

      if (this.player.techName_ !== 'Flash') this.player.pause()

      this.player.dispose()
      this.player = null

      this.$nextTick(() => {
        this.show = false
        this.$nextTick(() => {
          this.show = true
          this.$nextTick(() => {
            if (callback) callback()
          })
        })
      })
    }
  }
}
</script>

<style lang="sass">
  @import "~video.js/dist/video-js.css"

  .video-player > .video-js
    .vjs-big-play-button // 居中的大播放按钮
      top: 50%
      left: 50%
      transform: translate(-50%, -50%)

    .vjs-control-bar // 控制条
      height: 42px

      .vjs-progress-control // 进度条
        position: absolute
        width: 100%
        height: 10px
        top: -10px
        align-items: flex-end

        .vjs-progress-holder
          height: 5px
          margin: 0

          .vjs-play-progress
            background-color: #ff6e0b

            &:before
              display: none

        &:hover .vjs-progress-holder
          height: 10px

          .vjs-play-progress:before
            display: block
            top: 0
            height: 10px
            line-height: 10px
            font-size: 10px

      .vjs-icon-placeholder:before // 所有按钮
        line-height: 42px
        font-size: 2rem
</style>
