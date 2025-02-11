<template>
  <component :is="tag">
    <b-btn :disabled="RTC.roomState !== 2" size="sm" variant="link" v-b-tooltip.hover :title="screenTrackReadyState === 'live' ? $t('stop_share_screen') : $t('share_screen')" @click="switchScreenSharing">
      <Icon type="screen-share" height="22" />
    </b-btn>

    <!-- Chrome 屏幕共享插件安装提示 -->
    <b-modal id="SharingScreenExtensionModal" ref="SharingScreenExtensionModal" centered :lazy="true" :hide-header="true" :hide-footer="true" body-class="text-center">
      <h5 class="my-3 text-danger">Screen sharing plugin exception!</h5>
      <p>Please click the button to download and install the plugin.</p>
      <a class="btn btn-primary" href="https://chrome.google.com/webstore/detail/qnrtc-screen-capture/kelbkaklgngbpjmfopeiclhfmpgnkkpe" target="_blank">Installing the plugin</a>
    </b-modal>
  </component>
</template>

<script lang="ts">
import Vue from "vue";
import { mapState, mapGetters, mapMutations } from "vuex";
import * as QNRTC from "pili-rtc-web";

export default Vue.extend({
  props: {
    tag: {
      type: String,
      default: "div",
    }
  },

  data () {
    return {
      trackReadyStateInterval: 0,
      screenTrackReadyState: "",
    }
  },

  computed: {
    ...mapState("room", [
      "RTC",
    ])
  },

  methods: {
    // 开始屏幕共享
    async startSharing() {
      if (this.$browser.name === "chrome") {
        const isAvailable = await QNRTC.isChromeExtensionAvailable()
        if (!isAvailable) {
          this.$root.$emit("bv::show::modal", "SharingScreenExtensionModal");
          return
        }
      }

      try {
        var tracks = await QNRTC.deviceManager.getLocalTracks({
          screen: {
            enabled: true,
            tag: "screen",
          }
        });

        await this.RTC.publish(tracks);

        this.trackReadyStateInterval = window.setInterval(() => {
          for (const track of tracks) {
            this.screenTrackReadyState = track.mediaTrack.readyState;
          }
        }, 1000);
      } catch (error) {
        this.screenTrackReadyState === "";
      }
    },

    // 停止屏幕共享
    async stopSharing() {
      await this.RTC.unpublishWithTag("screen");
      this.screenTrackReadyState === "ended";
    },

    switchScreenSharing() {
      if (this.screenTrackReadyState !== "live"){
        this.startSharing();
        } else {
        this.stopSharing();
      }
    }
  },

  watch: {
    screenTrackReadyState: async function(state) {
      if (state === "ended") {
        await this.stopSharing();
        window.clearInterval(this.trackReadyStateInterval);
      }
    }
  },

  created () {

  },

  destroyed() {
    window.clearInterval(this.trackReadyStateInterval);
  }
})
</script>
