<template>
  <v-container class="fill-height">

  <v-btn>
    Button
  </v-btn>
  <v-progress-circular indeterminate color="primary"></v-progress-circular>
<!--    benlaizaizhe-->

    <v-dialog
      v-model="open"
      fullscreen
      :scrim="false"
      transition="dialog-bottom-transition"
    >
      <template v-slot:activator="{ props }">
        <v-btn
          color="primary"
          dark
          v-bind="props"
        >
          Open Dialog
        </v-btn>
      </template>
      <v-card>
        <v-toolbar
          dark
          color="primary"
        >
          <v-btn
            icon
            dark
            @click="clickMe"
          >
            <v-icon>mdi-close</v-icon>
          </v-btn>
          <v-toolbar-title>Settings</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-toolbar-items>
            <v-btn
              variant="text"
              @click="clickMe"
            >
              Save
            </v-btn>
          </v-toolbar-items>
        </v-toolbar>
        <v-list
          lines="two"
          subheader
        >
          <v-list-subheader>User Controls</v-list-subheader>
          <v-list-item title="Content filtering"
                       subtitle="Set the content filtering level to restrict apps that can be downloaded"></v-list-item>
          <v-list-item title="Password"
                       subtitle="Require password for purchase or use password to restrict purchase"></v-list-item>
        </v-list>
        <v-divider></v-divider>
        <v-list :items="tops"
                item-value="order"
                item-title="msg"
        ></v-list>
      </v-card>
    </v-dialog>
<!--    -->
  </v-container>

  <div>
    <v-container justify="center">
      <v-col sm="6" md="4" cols="auto">
        <v-btn @click="onQueryPermission">Request Audio Permission</v-btn>
      </v-col>
      <v-col sm="6" md="4" cols="auto">
        <v-btn :disabled="mediaRecorder == null" @click="onRecordSwitch">{{ inRecording ? "Stop" : "Start" }} Audio Recording</v-btn>
      </v-col>
      <v-col sm="6" md="4" cols="auto">
        <v-btn :disabled="audioURL == null">Upload</v-btn>
      </v-col>
      <audio v-if="audioURL != null" controls :src="audioURL"></audio>
    </v-container>
  </div>


</template>

<script setup>
import {onMounted, ref} from "vue";

const tops = [
  {
    order:1,
    name:"A",score:11
  },
  {order: 2,
    name:"B",score:1
  },
].map((v)=>{return {msg :`姓名：${v.name} 分数：${v.score}`,order :v.order}})

const notifications = ref(false)

const sound = ref(false)

const widgets = ref(false)
const open = ref(false)
const clickMe = ()=> {
  open.value =! open.value
}

onMounted(()=>{
  navigator.mediaDevices.getUserMedia({audio:true}).then((stream)=>{alert("get mic success")})
    .catch((err)=>{alert(`cannot get mic ${err}`)})
})




import fixWebmDuration from "fix-webm-duration"
import string from "string";
// 在未取得用户授权，没有相应的录音器实例存在
const mediaRecorder = ref<null | MediaRecorder>(null)
// 当前是否正在进行录音
const inRecording = ref(false)
// 录音结果的URL
const audioURL = ref<null | string>(null)
// 录音结果的原始Blob
const audioBlob = ref<null | Blob>(null)
// 录音开始时间
const startAt = ref(Date.now())

const onQueryPermission = () => {
  navigator.mediaDevices.getUserMedia({ audio: true }).then((stream) => {
    let recorder = new MediaRecorder(stream)

    recorder.onstart = () => {
      startAt.value = Date.now()
    }
    recorder.onstop = async () => {
      if (audioBlob.value != null) {
        const now = Date.now()
        const duration = now - startAt.value
        const blob = await fixWebmDuration(audioBlob.value, duration)
        const blobOgg = new Blob([blob], { type: "audio/ogg; codecs=opus" })
        audioURL.value = window.URL.createObjectURL(blob)
        audioBlob.value = blobOgg
        console.log(`audio record done URL ${audioURL.value}`)
      }
    }

    recorder.ondataavailable = (ev) => {
      audioBlob.value = ev.data
    }
    mediaRecorder.value = recorder
  })
  console.log("media recorder create done")
}

const onRecordSwitch = () => {
  if (inRecording.value) {
    mediaRecorder.value?.stop()
    inRecording.value = false
  } else {
    audioBlob.value = null
    audioURL.value = null
    mediaRecorder.value?.start()
    inRecording.value = true
  }
  console.log(`now record state is ${mediaRecorder.value?.state}`)
}



</script>

<style scoped>

</style>
