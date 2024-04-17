<template>
  <div>
    <input type="file" @change="processAudio" accept="audio/*">
    <input type="range" min="0" max="100" v-model="gainValue" />
    <button @click="playAudio">播放音频</button>
    <button @click="stopAudio">停止播放</button>
  </div>
</template> 

<script>
export default {
  name: 'lowAudio',
  props: {
    msg: String
  },
  data() {
    return {
      audioContext: null,
      source: null,
      audioBuffer: null,
      gainValue: 30,
    };
  },
  methods: {
    processAudio(event) {
      const file = event.target.files[0];
      const reader = new window.FileReader();
      reader.readAsArrayBuffer(file);

      reader.onloadend = () => {
        // 创建 audio context
        this.audioContext = new (window.AudioContext || window.webkitAudioContext)();
        this.audioContext.decodeAudioData(reader.result, buffer => {
          this.audioBuffer = buffer;

          // 如果已经存在 source，需要先将其移除
          if (this.source) {
            this.source.disconnect();
          }
        });
      };
    },
    playAudio() {
      // 创建 source
      this.source = this.audioContext.createBufferSource();
      this.source.buffer = this.audioBuffer;

      // 创建 biquad filter
      const biquadFilter = this.audioContext.createBiquadFilter();
      // 设置 lofi 参数
      biquadFilter.type = "lowshelf";
      biquadFilter.frequency.value = 1000;

      // 从滑块获取增益值
      biquadFilter.gain.value = this.gainValue;

      // 连接 source --> biquad --> destination
      this.source.connect(biquadFilter);
      biquadFilter.connect(this.audioContext.destination);

      this.source.start(0);
    },
    stopAudio() {
      if (this.source) {
        this.source.stop();
        this.source.disconnect();
        this.source = null;
      }
    },
  },
};
</script>