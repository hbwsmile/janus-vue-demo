<template>
  <div id="app">
    <div class="container">
      <div class="columns">
        <div class="column is-half">
          <player
            ref="player"
            :class="{'is-playing' : isPlaying}"
            :rtspSrc="rtspSrc"/>
        </div>
      </div>
      <div class="columns">
        <div class="column is-half">
          <form>
            <b-field>
              <b-autocomplete
                class="player-url"
                v-model="rtspSrc"
                title="Для ввода адреса, остановите воспроизведение"
                :data="serverList"
                @keyup.enter.native="isPlaying ? $refs.player.stopStream() : $refs.player.startStream()"
                placeholder="rtsp://..."
                icon-pack="fas"
                icon="video"
                @select="option => selected = option">
              </b-autocomplete>
              <p class="control">
                <b-button
                  v-if="$refs.player"
                  @click="isPlaying ? $refs.player.stopStream() : $refs.player.startStream()"
                  icon-pack="fas"
                  :icon-right="isPlaying ? 'stop' :  'play'"
                  :type="isPlaying ? 'is-dark' : 'is-success'" />
              </p>
            </b-field>
          </form>
        </div>
      </div>

      <div
        v-if="$route.query.debug"
        class="columns">
        <div class="column logs-panel" ref="consolePanel">
          <b-tabs
            expanded
            v-model="logTabsActive">
            <b-tab-item
              v-for="(logType) in ['debug', 'error', 'info']"
              :key="logType"
              :label="logType">
                <div
                  :class="logType"
                  class="logs-panel-scroll">
                  <template
                    v-for="(log, key) in console[logType]">
                    <p
                      class="log-item"
                      :key="key"
                      :class="log.type">
                      <b-tag
                        :type="`is-${logsTypeClasses[log.type]}`"
                        @close="isTag2Active = false">
                          {{log.type}}
                      </b-tag>
                      {{log.message}}
                    </p>
                  </template>
                </div>
              </b-tab-item>
          </b-tabs>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import player from './components/AppPlayer.vue'

export default {
  name: 'app',
  components: {
    player
  },
  data: () => ({
    appConsole: [],
    showLogs: false,
    rtspSrc: "",
    logTabsActive: 0,
    logsTypeClasses: {
      debug: 'dark',
      error: 'danger',
      info: 'info'
    },
    selected: null
  }),
  watch:{
    console(){
      this.$refs.consolePanel.scrollTop = this.$refs.consolePanel.scrollHeight
    }
  },
  computed: {
    isPlaying:{
      get(){
        return this.$store.state.isPlaying
      },
      set(value){
        this.$store.commit('SET_PLAYING_STATE', value)
      }
    },
    serverList:{
      get(){
        return this.$store.state.RTSPServerList
      },
      set(value){
        this.$store.commit('SET_RTSP_SERVER_LIST', value)
      }
    },
    console:{
      get(){
        return this.$store.state.logs
      },
      set(value){
        this.$store.commit('SET_LOG', value)
      }
    }
  },
  mounted() {
    if (this.$route.query.clear) {
      this.$store.commit('CLEAR_RTSP_SERVER_LIST')
    }

    if (this.serverList.length && !this.$store.state.activeRTSPUrl && !this.rtspSrc) {
      this.$store.state.activeRTSPUrl = this.serverList[this.serverList.length-1]
      this.rtspSrc = this.serverList[this.serverList.length-1]
    }
  },

  methods: {

  }
}
</script>
<style lang="scss">
  @import "bulma/sass/utilities/initial-variables.sass";
  @import "bulma/sass/utilities/derived-variables.sass";


  .columns{
    justify-content: center
  }
  .player{
    margin-top: 30px;
    position: relative;
    padding-bottom: 56.25%; /* 16:9 */
    padding-top: 25px;
    height: 0;
    background: #000;
    video {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
    }
  }
  .player-url{
    width: 100%;
  }
  form{
    width: 100%;
    display: flex;
    flex-direction: column;
    margin-top: 20px;
  }
  .logs-panel{

    &-scroll{
      overflow-y: scroll;
      height: 200px;
    }
    .log-item{
      margin-bottom: 5px;
      &.error{
        color: $red;
      }
      &.info{
        color: $info;
      }
      &.debug{
        color: $grey-light;
      }
    }
  }
</style>