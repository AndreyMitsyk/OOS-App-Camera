<template>
  <div class="container bg">
    <div class="card">
      <div class="card-image">
        <img :src="streamUrl" onerror="static/img/not_available.jpg" class="stream img-responsive">
      </div>
      <div class="card-header">
        <div class="card-title h5">Live Stream</div>
        <!-- <div class="card-subtitle text-gray">...</div> -->
      </div>
      <!-- <div class="card-body">
        ...
      </div>
      <div class="card-footer">
        <button class="btn btn-primary">...</button>
      </div> -->
    </div>
    <div class="card">
      <!-- <div class="card-image">
        <img :src="streamUrl" onerror="static/img/not_available.jpg" class="stream img-responsive">
      </div> -->
      <div class="card-header">
        <div class="card-title h5">Timelapse {{bTimelapse?'(Recording)':''}}</div>
        <div class="card-subtitle text-gray">Periodic snapshots will be sent directly to your <strong>Dropbox</strong></div>

      </div>
      <div class="card-body">
        <div class="input-group">
          <!-- <span class="input-group-addon"><a>How to get the Token?</a></span> -->
          <input type="text" class="form-input" v-model="token" placeholder="Dropbox App Token" >
          <button class="btn btn-primary input-group-btn"
          :class="{loading: isLoading}"
          @click="saveToken">Save</button>
        </div>
        <div class="card-subtitle text-gray"><a href="https://onion.io/onionos/Timelapse" target="_blank">How do I get an App Token?</a></div>
      </div>
      <div class="card-footer">
        <button class="btn btn-primary btn-block"
        @click="toggleRecording"
        :class="{loading: isLoading}"
        >{{bTimelapse?'Stop Recording':'Start Recording'}}</button>
      </div>
    </div>
    <!-- <header class="navbar">
      <section class="navbar-section">
      </section>
      <section class="navbar-section my-1">
        <a @click="startStream" class="btn mr-1" :disabled="!bReady">{{streamButtonText}}</a>
      </section>
    </header>
    <div class="container grid-lg">
      <div class="card my-2">
        <div class="card-body">
          <img class="centered" :src="streamUrl" onerror="static/img/not_available.jpg">
        </div>
      </div>
      <div class="card">
        <div class="card-header">
          <div class="card-title h5">...</div>
          <div class="card-subtitle text-gray">...</div>
        </div>
        <div class="card-body">
          HELLO THERE
        </div>
        <div class="card-footer">
          <button class="btn btn-primary">...</button>
        </div>
      </div>
    </div> -->
  </div>
</template>

<script>
import OnionCDK from '@/OnionCDK.js'

var devMode = process.env.NODE_ENV === 'development'
var devHost = process.env.DEV_HOST

export default {
  name: 'App',
  components: {},
  data () {
    return {
      port: 8080,
      bStreaming: false,
      bTimelapse: false,
      bReady: false,
      isLoading: true,
      token: ''
    }
  },
  methods: {
    created () {
      console.log(window.location.hostname)
      this.startStream()
    },
    startStream () {
      if (this.bStreaming) {
        // stream is currently running, stop it
        OnionCDK.service('mjpg-streamer', 'stop')
      } else {
        // stream is stopped, start it
        OnionCDK.service('mjpg-streamer', 'start')
      }
      // TODO: need to find a place to configure mjpg-streamer
      // OnionCDK.sendCmd('getBatteryLevels', [])
    },
    saveToken () {
      this.isLoading = true
      OnionCDK.sendCmd('/www/apps/oos-app-timelapse-camera/save-token.sh', [this.token])
    },
    toggleRecording () {
      this.isLoading = true
      if (this.bTimelapse) {
        OnionCDK.service('oos-app-timelapse', 'stop')
      } else {
        OnionCDK.service('oos-app-timelapse', 'start')
      }
    }
  },
  mounted () {
    OnionCDK.onService = function (name, command, result) {
      // console.log(name, command, result)
      this.isLoading = false
      switch (name) {
        case 'oos-app-timelapse':
          if (command === 'list' && typeof result !== 'undefined') {
            this.bTimelapse = result
          } else if (command === 'start' && typeof result !== 'undefined') {
            this.bTimelapse = result
          } else if (command === 'stop' && typeof result !== 'undefined') {
            this.bTimelapse = !result
          }
          break
        case 'mjpg-streamer':
          if (command === 'list' && typeof result !== 'undefined') {
            this.bStreaming = result
            this.bReady = true
          } else if (command === 'start' && typeof result !== 'undefined') {
            this.bStreaming = result
          } else if (command === 'stop' && typeof result !== 'undefined') {
            this.bStreaming = !result
          }
          // TODO: add handling for STOP and START commands
          break
        default:
          break
      }
    }.bind(this)

    OnionCDK.onCmd = function (command, result) {
      this.isLoading = false
      switch (command) {
        case '/www/apps/oos-app-timelapse-camera/save-token.sh':
          OnionCDK.sendToast('Token Saved')
          break
        case 'cat':
          this.token = result || ''
          break
        default:
          break
      }
    }.bind(this)

    OnionCDK.onInit = function () {
      // check if streaming service is running
      OnionCDK.service('mjpg-streamer', 'list')
      OnionCDK.service('oos-app-timelapse', 'list')
      OnionCDK.sendCmd('cat', ['/www/apps/oos-app-timelapse-camera/token.txt'])
    }
    OnionCDK.init()
  },
  computed: {
    streamUrl () {
      return `http://${devMode ? devHost : window.location.hostname}:${this.port}/?action=stream`
    },
    streamButtonText () {
      return (this.bStreaming ? 'Stop Stream' : 'Start Stream')
    }
  }
}
</script>

<style>
html, body {
    height: 100% !important;
    height: 100%;
}
body {
  margin: 0;
  padding: 0;
  background-color: white;
}
.navbar {
  background-color: #9b59b6;
}
.bg {
  /* background-color: #95a5a6; */
  height:100%;
}

.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  height: auto;
}

.stream {
  width: 100%;
}

.card {
  margin-top: 40px;
  width: 100%;
  max-width: 800px;
}
</style>
