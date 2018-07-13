<template>
    <canvas ref="visuals" :width="width" :height="height"></canvas>
</template>

<script>
/* eslint-disable */
export default {
  name: "Visualization",
  props: ["analyser", "width", "height"],
  data : function(){
    return {
      a: this.analyser
    }
  },
  mounted() {
    console.log(this.a);
    if(this.a){
      this.bufferLength = this.a.frequencyBinCount;
      this.dataArray = new Uint8Array(this.bufferLength);
      this.a.fftSize = 2048;
      this.a.getByteTimeDomainData(this.dataArray);
      this.canvasCtx = this.$refs["visuals"].getContext("2d");
      this.canvasCtx.clearRect(0, 0, this.width, this.height);
      this.draw();
    }
  },
  beforeDestroy() {
    this.a = null
  },
  methods: {
    draw: function(canvasCtx) {
      var analyser = this.a;
      var canvasCtx = this.canvasCtx;
      var width = this.width;
      var height = this.height;
      var dataArray = this.dataArray;
      var bufferLength = this.bufferLength;

      if(!analyser) return;

      var drawVisual = requestAnimationFrame(this.draw);
      analyser.getByteTimeDomainData(dataArray);
      canvasCtx.fillStyle = "rgb(163, 215, 224)";
      canvasCtx.fillRect(0, 0, width, height);
      canvasCtx.lineWidth = 4;
      canvasCtx.strokeStyle = "rgb(255, 255, 255)";

      canvasCtx.beginPath();
      var sliceWidth = width * 1.0 / bufferLength;
      var x = 0;
      for (var i = 0; i < bufferLength; i++) {
        var v = dataArray[i] / 128.0;
        var y = v * height / 2;

        if (i === 0) {
          canvasCtx.moveTo(x, y);
        } else {
          canvasCtx.lineTo(x, y);
        }

        x += sliceWidth;
      }
      canvasCtx.lineTo(this.$refs["visuals"].width, this.$refs["visuals"].height / 2);
      canvasCtx.stroke();
    }
  }
};
</script>