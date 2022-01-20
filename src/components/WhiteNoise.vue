<template>
  <p>
    🔊<input
      id="vol"
      v-model.number="volume"
      type="range"
      min="0"
      max="1"
      step="0.01"
      v-on:change="changeVolume()"
    />
    <br />
    <button id="play" v-on:click="play()">▶️</button>
    <button id="stop" v-on:click="stop()">⏸️</button>
  </p>
</template>
<script>
export default {
  name: "WhiteNoise",
  props: {},
  data() {
    return {
      node: null,
      bufferSize: 0, // 256 - 16384
      numberOfInputChannels: 1,
      numberOfOutputChannels: 1,
      gainNode: null,
      ctx: null,
      volume: 0.01,
      average: 0.0,
      deviation: 1.0,
    };
  },
  mounted: function () {
    this.player();
  },
  methods: {
    play() {
      this.node.connect(this.gainNode);
      this.gainNode.connect(this.ctx.destination);
    },
    stop() {
      this.node.disconnect();
    },
    changeVolume() {
      this.gainNode.gain.value = this.volume;
    },
    player() {
      this.ctx = new AudioContext();

      this.node = this.ctx.createScriptProcessor(
        this.bufferSize,
        this.numberOfInputChannels,
        this.numberOfOutputChannels
      );

      this.gainNode = this.ctx.createGain();
      this.gainNode.gain.value = this.volume;

      this.node.onaudioprocess = function (e) {
        var data = e.outputBuffer.getChannelData(0);

        for (var i = 0; i < data.length; i++) {
          data[i] = xRandomNormal(this.average, this.deviation);
        }
      }.bind(this);

      const xRandomNormal = function (average, deviation) {
        // http://www.kogures.com/hitoshi/webtext/stat-random/index.html
        var z01 = 0;
        for (var i = 1; i <= 12; i++) {
          z01 = z01 + Math.random();
        }
        z01 = z01 - 6;
        return average + z01 + deviation * z01;
      };
    },
  },
};
</script>

<style scoped>
</style>