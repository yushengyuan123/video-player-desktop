<template>
  <div
    :class="['control-container', idleClass]"
  >
    <div class="control-container-inner">
      <div class="control-button-area">
        <el-row align="middle" style="height: 100%">
          <el-col :span="5">
            <el-row align="middle">
              <div class="control-volume-container">
                <icon-collect name="volume"/>
              </div>
              <video-progress
                :percentage="curVolume"
                :width="65"
                @change="volumeChange"
              />
            </el-row>
          </el-col>
          <el-col :span="14">
            <el-row align="middle" justify="center">
              <icon-collect name="back"/>
              <div class="video-status-con" @click="switchVideoStatus">
                <icon-collect v-if="!videoStatus" name="continue"/>
                <icon-collect v-else name="pause"/>
              </div>
              <icon-collect name="forward"/>
            </el-row>
          </el-col>
          <el-col :span="5">
            <el-row justify="end">
              <div class="video-menu-operate-con" @click="backHomePage">
                <icon-collect name="back-home"/>
              </div>
              <div class="video-menu-operate-con" @click="settingsVisible">
                <icon-collect name="setting"/>
              </div>
              <div class="video-menu-operate-con" @click="switchMode">
                <icon-collect v-if="!videoFullScreenMode" name="full-screen"/>
                <icon-collect v-else name="close-full-screen"/>
              </div>
            </el-row>
          </el-col>
        </el-row>
      </div>
      <div class="control-progress-area">
        <el-row align="middle">
          <el-col :span="3">{{ currentTime }}</el-col>
          <el-col :span="18">
            <video-progress
              :percentage="updatePercentage"
              @change="videoProgressChange"
            />
          </el-col>
          <el-col :span="3">
            <el-row justify="end">
              {{ duration }}
            </el-row>
          </el-col>
        </el-row>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import {
  defineComponent,
  computed
} from "vue"
import { useIdle } from "@vueuse/core";
import { useControl } from "./use-control";
import Progress from "./Progress.vue"
import { useVideoInfo } from "../../store/videoInfo"
import { storeToRefs } from "pinia"
import { convertSecondsToTime } from "../../utils"

export default defineComponent({
  name: "video-control-area",
  components: {
    [Progress.name]: Progress
  },
  setup() {
    const videoInfoStore = useVideoInfo()
    const { updateMode } = videoInfoStore
    const { videoFullScreenMode } = storeToRefs(videoInfoStore)
    const {
      videoDuration,
      videoCurrentTime,
      videoStatus,
      videoVolume
    } = storeToRefs(videoInfoStore)
    const curVolume = videoVolume.value / 100
    
    const {
      switchVideoStatus,
      leftMenuVisible,
      volumeChange,
      videoProgressChange,
      settingsVisible,
      backHomePage
    } = useControl()

    const switchMode = () => {     
      updateMode(!videoFullScreenMode.value)
    }

    const { idle } = useIdle(4000)

    const idleClass = computed(
      () => {
        if (idle.value) {
          return 'control-user-idle'
        }
        return ''
      }
    )

    const updatePercentage = computed(() => {
      return parseFloat((videoCurrentTime.value / videoDuration.value).toFixed(4))
    })

    const duration = computed(
      () => convertSecondsToTime(videoDuration.value)
    )

    const currentTime = computed(
      () => convertSecondsToTime(videoCurrentTime.value)
    )

    return {
      switchMode,
      videoStatus,
      duration,
      currentTime,
      idleClass,
      curVolume,
      videoFullScreenMode,
      switchVideoStatus,
      updatePercentage,
      leftMenuVisible,
      settingsVisible,
      videoProgressChange,
      volumeChange,
      backHomePage,
    }
  }
})
</script>

<style scoped lang="less">
.control-container {
  @width: 520px;
  position: absolute;
  left: 50%;
  margin-left: -(@width / 2);
  bottom: 100px;
  height: 80px;
  width: @width;
  opacity: 0.7;
  z-index: 20000;
  border-radius: 6px;
  overflow: hidden;


  &.control-user-idle {
    transition: opacity .8s linear;
    opacity: 0;
  }

  .control-container-inner {
    height: 100%;
    width: 100%;
    display: flex;
    flex-direction: column;
    transition: all .3s ease-in-out;
    padding-top: 5px;

    .control-button-area {
      height: 45px;
      width: 100%;
      padding-left: 10px;
      padding-right: 10px;
      box-sizing: border-box;

      .control-volume-container {
        height: 100%;
        margin-right: 8px;
      }

      .video-status-con {
        display: flex;
        align-items: center;
        justify-content: center;
        height: 40px;
        width: 40px;
      }

      .video-menu-operate-con {
        margin-left: 10px;
      }
    }

    .control-progress-area {
      flex: 1;
      height: 100%;
      width: 100%;
      font-size: 12px;
      padding-left: 10px;
      padding-right: 10px;
      padding-bottom: 13px;
      box-sizing: border-box;
    }
  }
}

.control-container:after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: -1;
  filter: blur(5px);
}


</style>
