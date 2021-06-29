<template>
  <section class="vueVideo">
    <div
      v-video-player:myVideoPlayer="playerOptions"
      class="video-player-box"
      :playsinline="playsinline"
      @play="onPlayerPlay($event)"
      @pause="onPlayerPause($event)"
      @ended="onPlayerEnded($event)"
      @loadeddata="onPlayerLoadeddata($event)"
      @waiting="onPlayerWaiting($event)"
      @playing="onPlayerPlaying($event)"
      @timeupdate="onPlayerTimeupdate($event)"
      @canplay="onPlayerCanplay($event)"
      @canplaythrough="onPlayerCanplaythrough($event)"
      @ready="playerReadied"
      @statechanged="playerStateChanged($event)"
    ></div>
  </section>
</template>

<script>
import 'video.js/dist/video-js.css'
import 'vue-video-player/src/custom-theme.css'
export default {
  props: {
    playerOptions: {
      type: Object,
      default: null
    }
  },
  data() {
    return {
                playsinline: true
    }
  },
  mounted() {
    console.log('this is current player instance object', this.myVideoPlayer)
  },
  methods: {
    // 监听播放
    onPlayerPlay(player) {
      // console.log('player play!', player)
    },
    // 监听暂停
    onPlayerPause(player) {
      // console.log('player pause!', player)
    },
    // 监听停止
    onPlayerEnded(player) {
      // console.log('player ended!', player)
    },
    // 监听加载完成
    onPlayerLoadeddata(player) {
      // console.log('player Loadeddata!', player)
    },
    // 监听视频缓存等待
    onPlayerWaiting(player) {
      // console.log('player Waiting!', player)
    },
    // 监听视频暂停后播放
    onPlayerPlaying(player) {
      // console.log('player Playing!', player)
    },
    // 监听视频播放时长更新
    onPlayerTimeupdate(player) {
      // console.log('player Timeupdate!', player.currentTime())
    },
    onPlayerCanplay(player) {
      console.log('player Canplay!', player)
    },
    onPlayerCanplaythrough(player) {
      // console.log('player Canplaythrough!', player)
    },
    // 监听状态改变
    playerStateChanged(playerCurrentState) {
      // console.log('player current update state', playerCurrentState)
    },
    // 监听播放器准备就绪
    playerReadied(player) {
      // console.log('example 01: the player is readied', player)
    }
  }
}
</script>

<style lang="scss" scoped>
.video_win {
      width: 1110px;
      height: 585px;
      -webkit-background-size: 100%;
      background-size: 100%;
      &.mini_win {
        width: 600px;
        height: 295px;
        position: fixed;
        right: 10px;
        bottom: 30px;
        z-index: 9999;
      }
    }
</style>
