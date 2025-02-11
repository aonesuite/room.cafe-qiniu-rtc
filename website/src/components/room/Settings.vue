<template>
  <b-modal class="modal-room-settings" id="RoomSettingsModal" ref="RoomSettingsModal" centered :lazy="true" hide-footer hide-header @show="modalShow()">
    <form v-on:submit.prevent="onSubmit">

      <el-tabs class="tabs" :stretch="true" v-model="tabActive">
        <el-tab-pane :label="$t('room_settings.general')" name="general">
          <div class="form-group">
            <label for="video-device">{{ $t('room_settings.camera') }}</label>
            <el-select class="d-block" id="video-device" :placeholder="$t('room_settings.placeholder_select_camera')" v-model="settings.currentVideoInputDeviceID">
              <el-option v-for="device in deviceInfoList.filter((info) => info && info.kind === 'videoinput')" :key="device.deviceId" :label="device.label" :value="device.deviceId"></el-option>
            </el-select>
          </div>

          <div class="form-group">
            <label for="microphone-device">{{ $t('room_settings.microphone') }}</label>
            <el-select class="d-block" id="microphone-device" :placeholder="$t('room_settings.placeholder_select_microphone')" v-model="settings.currentAudioInputDeviceID">
              <el-option v-for="device in deviceInfoList.filter((info) => info && info.kind === 'audioinput')" :key="device.deviceId" :label="device.label" :value="device.deviceId"></el-option>
            </el-select>
          </div>

          <div class="form-group">
            <label for="speaker-device">{{ $t('room_settings.speakers') }}</label>
            <el-select disabled class="d-block" id="speaker-device" :placeholder="$t('room_settings.placeholder_select_speaker')" v-model="settings.currentAudioOutputDevice">
              <el-option v-for="device in deviceInfoList.filter((info) => info && info.kind === 'audiooutput')" :key="device.deviceId" :label="device.label" :value="device.deviceId"></el-option>
            </el-select>
          </div>
        </el-tab-pane>

        <el-tab-pane :label="$t('room_settings.bandwidth')" name="bandwidth">
          <div class="form-group">
            <label for="incoming-video">{{ $t('room_settings.incoming_video') }}</label>
            <el-select class="d-block" id="incoming-video" :placeholder="$t('room_settings.placeholder_select_resolution')" v-model="settings.clarity">
              <el-option v-for="(clarity, key) in clarities" :key="key" :label="$t(`clarity.${key}`)" :value="key"></el-option>
            </el-select>
          </div>
        </el-tab-pane>

        <el-tab-pane :label="$t('room_settings.langs')" name="langs">
          <div class="form-group">
            <label for="lang">{{ $t('room_settings.langs') }}</label>
            <el-select class="d-block" id="lang" :placeholder="$t('room_settings.placeholder_select_langs')" v-model="$i18n.locale" @click="changeLang($i18n.locale)">
              <el-option v-for="(label, lang) in langs" :key="lang" :label="label" :value="lang"></el-option>
            </el-select>
          </div>
        </el-tab-pane>
      </el-tabs>

      <div class="clearfix text-right">
        <b-btn size="sm" variant="outline-secondary" class="mr-2" @click="$root.$emit('bv::hide::modal', 'RoomSettingsModal')">{{ $t('room_settings.cancel') }}</b-btn>
        <b-btn type="submit" size="sm" variant="outline-primary" :disabled="isSubmitting">{{ $t('room_settings.done') }}</b-btn>
      </div>
    </form>
  </b-modal>
</template>

<script lang="ts">
import Vue from 'vue';
import { mapState, mapActions } from 'vuex';
import * as QNRTC from 'pili-rtc-web';
import * as Clarity from '../../constants/clarity';
import { langs } from '../../locales';

export default Vue.extend({

  data() {
    return {
      tabActive: "general",
      langs: langs,
      deviceInfoList: [] as MediaDeviceInfo[],
      settings: {
        currentAudioOutputDevice: 'default',
        currentAudioInputDeviceID: 'default',
        currentVideoInputDeviceID: '',
        clarity: "SD"
      },
      clarities: {},
      isSubmitting: false
    }
  },

  computed: {
    ...mapState("room", [
      "roomInfo",
      "RTC"
    ]),

  },

  methods: {
    modalShow() {

    },

    // 发布本地音视频
    async publish() {
      if (this.RTC.roomState != QNRTC.RoomState.Connected) return;

      const clarity = Clarity.getClarity(this.settings.clarity as Clarity.ClarityType);
      var tracks = await QNRTC.deviceManager.getLocalTracks({
        audio: {
          enabled: true,
          tag: "master",
          deviceId: this.settings.currentAudioInputDeviceID
        },
        video: {
          enabled: true,
          tag: "master",
          deviceId: this.settings.currentVideoInputDeviceID,
          width: clarity.width,
          height: clarity.height,
          bitrate: clarity.bitrate,
        }
      });

      tracks.map(track => track.setMaster(true));
      await this.RTC.unpublishWithTag("master");
      await this.RTC.publish(tracks);
      this.$root.$emit('bv::hide::modal', 'RoomSettingsModal');
    },

    async onSubmit () {
      this.isSubmitting = true
      await this.publish()
      this.isSubmitting = false
    },

    changeLang(lang: string) {
      this.$i18n.locale = lang;
      localStorage.setItem("locale", lang);
      if (this.$refs.langSwitchPopover === undefined) return;
      const langSwitchPopover = this.$refs.langSwitchPopover as any;
      langSwitchPopover.$emit('close');
    }
  },

  created () {
    this.deviceInfoList = QNRTC.deviceManager.deviceInfo || [] as MediaDeviceInfo[];

    QNRTC.deviceManager.on('device-update', (deviceInfoList: MediaDeviceInfo[]) => {
      this.deviceInfoList = deviceInfoList
    })

    const videoInputDevices = this.deviceInfoList.filter((info) => info && info.kind === 'videoinput')
    if (videoInputDevices.length === 1 && videoInputDevices[0].deviceId) {
      this.settings.currentVideoInputDeviceID = videoInputDevices[0].deviceId
    }

    this.clarities = Clarity.clarities;
  },

  mounted () {

  }
})
</script>
