<template>
  <p>
    🔊<input
      id="vol"
      v-model.number="volume"
      type="range"
      min="0"
      max="0.1"
      step="0.001"
      v-on:change="changeVolume()"
    />
    <br />
    <button v-if="playing" id="stop" v-on:click="stop()">⏸️</button>
    <button v-else id="play" v-on:click="play()">▶️</button>
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
      playing:false,
    };
  },
  methods: {
    play() {
      this.player();
      this.node.connect(this.gainNode);
      this.gainNode.connect(this.ctx.destination);
      this.playing=true;
    },
    stop() {
      this.node.disconnect();
      this.playing=false;
    },
    changeVolume() {
      this.gainNode.gain.value = this.volume;
    },
    player() {
      if(this.ctx) return false;
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

      return true;
    },
  },
};
</script>

<style scoped>
</style>