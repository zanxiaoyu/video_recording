<template>
  <div>
    <div>当前录制窗口为：<span>{{ sourceId }}</span></div>
    <button @click="startRecord">开始录制</button>
    <button>暂停</button>
    <button>恢复</button>
    <button @click="stopRecord">停止录制</button>
    <button @click="saveFile">保存</button>
    <div class="con">
      <div class="item" v-for="item of data" :key="item.id" @click="choose(item.id)">{{ item.name }}</div>
    </div>
  </div>
</template>

<script>
const electron = require("electron");
const desktopCapturer = electron.desktopCapturer;
const fs = require("fs");

export default {
  data() {
    return {
      data: [],
      sourceId: "",
      _screenStream: null,
      _stream: null,
      _audioStream: null,
      _recorder: null,
      recordedBlobs: []
    }
  },
  created() {
    desktopCapturer.getSources({types: ['window', 'screen']}, (error, sources) => {
      if(error) throw error;
      console.log(sources);
      this.data = sources
    })
  },
  methods: {
    choose(id) {
      this.sourceId = id
      this.init()
    },
    async init() {
      this._screenStream = await navigator.mediaDevices.getUserMedia({
          audio: false,
          video: {
              mandatory: {
                  chromeMediaSource: 'desktop',
                  chromeMediaSourceId: this.sourceId,
                  minWidth: 1920,
                  maxWidth: 1920,
                  minHeight: 1080,
                  maxHeight: 1080
              }
          }
      });
      this._stream = new MediaStream();
      await this.attachAudioStream();
      this._screenStream.getTracks().forEach(t => this._stream.addTrack(t));
      this._recorder = new MediaRecorder(this._stream, {mimeType: "video/webm"});
      this._recorder.ondataavailable = async e => {
          console.log(e);
          this.recordedBlobs = [];
          this.recordedBlobs.push(e.data);
      };
    },
    // 获取音频
    async attachAudioStream() {
        this._audioStream = await navigator.mediaDevices.getUserMedia({video: false, audio: true});
        this._audioStream.getAudioTracks().forEach(value => this._stream.addTrack(value));
    },
    // 开始录制
    startRecord() {
      console.log(this._stream);
       this._recorder && this._recorder.start()
    },
    // 停止录制
    stopRecord(){
        this._recorder && this._recorder.state !== "inactive" && this._recorder.stop();
    },
    // 暂停
    pause() {
      this._recorder.pause()
    },
    // 恢复
    resume() {
      this._recorder.resume()
    },
    saveFile() {
      const blob = new Blob(this.recordedBlobs, { type: 'video/mp4' });
        let reader = new FileReader();
        reader.onload = () => {
            let buffer = new Buffer(reader.result);
            console.log(buffer);
            fs.writeFile('demo.mp4', buffer, {}, (err, res1) => {
              if (err) return console.error(err);
              this.recordedBlobs = []
            });
        };
        reader.onerror = err => console.error(err);
        reader.readAsArrayBuffer(blob);
    }
  }
}
</script>

<style lang="scss">
  .con {
    overflow-y: overlay;
    .item {
      padding: 10px;
      border: 1px solid #ccc;
      margin:  10px 0;
      cursor: pointer;
    } 
  }
  
</style>