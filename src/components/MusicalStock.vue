<template>
  <div class="main">
    <h1>{{ symbol }} - ${{currentPrice}}</h1>
    <div class="chart-container">
      <line-chart :min="this.minValue" :max="maxValue" :data="chartData"></line-chart>
    </div>
    <div class="panel">
      <button v-if="!playing" class="play" v-on:click="playOscillator"></button>
      <button v-if="playing" class="stop" v-on:click="stopOscillator"></button>
      <div class="selector">
        <div v-bind:class="[{ active: oscType=='triangle' && !playing }, 'triangle']" v-on:click="changeOscType('triangle')"></div>
        <div v-bind:class="[{ active: oscType=='square' && !playing }, 'square']" v-on:click="changeOscType('square')"></div>
        <div v-bind:class="[{ active: oscType=='sawtooth' && !playing }, 'sawtooth']" v-on:click="changeOscType('sawtooth')"></div>
        <div v-bind:class="[{ active: oscType=='sine' && !playing }, 'sine']" v-on:click="changeOscType('sine')"></div>
      </div>
      <div class="canvas-wrapper">
        <Visualization v-if="playing" :analyser="analyser" :width="w" :height="h"/>
      </div>
    </div>
  </div>
</template>

<script>
/* eslint-disable */
import Vue from 'vue'
import Visualization from './Visualization';
import VueChartkick from 'vue-chartkick'
import _ from 'lodash'

Vue.use(VueChartkick)

var SCALE_C = ["0","2","4","5","7","9","11","12","14","16","17","19","21","23","24","26","28","29","31","33",
"35","36","38","40","41","43","45","47","48","50","52","53","55","57","59","60","62","64","65","67","69","71",
"72","74","76","77","79","81","83","84","86","88","89","91","93","95","96","98","100","101","103","105","107",
"108","110","112","113","115","117","119","120","122","124","125","127"]

export default {
  name: 'MusicalStock',
  components: {
    VueChartkick,
    Visualization
  },
  props: ['symbol','prices','dates','minValue','maxValue', 'chartData'],
  data(){
    return {
      playing: false,
      ac: null,
      oscType: "sine",
      analyser: null,
      w: null,
      h: null
    }
  },
  computed : {
    currentPrice : function(){
      console.log(this.chartData[this.chartData.length -1]);
      return parseFloat(this.chartData[this.chartData.length -1][1]).toFixed(2);
    }
  },
  mounted(){
    this.onWindowsResize();
    window.addEventListener('resize', this.onWindowsResize);
  },
  beforeDestroy(){
    window.removeEventListener('resize', this.onWindowsResize);
  },
  methods: {
    playOscillator(){
      var midiArray = this.getMidiToFreqArray();
      var scale = this.getScale(midiArray, SCALE_C);
      var octaves = 3
      var startNote = 36
      var endNote = startNote + (octaves*8);
      
      var ac = this.ac = new (window.AudioContext || window.webkitAudioContext);
      this.analyser = ac.createAnalyser();
      var source;
      var gainNode = null;
      var oscNode = null;
      var startTime  = 0;
      var stopTime  = 0;
      var fadeOutTime  = 0;
      var freq = 0;
      var closePrice;
      var midiRangedValue;
      var nodeTypes = ['triangle', 'sine'];
      var combine = true;
      
      for(var i=0; i<this.prices.length; i++){
        startTime = i/5;
        fadeOutTime = startTime + 0.3;
        closePrice = this.prices[i][3];

        midiRangedValue = _.ceil((((closePrice - this.minValue) * (endNote - startNote)) / (this.maxValue - this.minValue)) + startNote);
        freq = scale[midiRangedValue];
      
        //we stop the oscnode after gain fades
        stopTime = fadeOutTime + 1.2;

        gainNode = ac.createGain();
        oscNode = ac.createOscillator();
        
        if(combine){
          oscNode.type = nodeTypes[Math.floor(Math.random() * nodeTypes.length)];
        } else {
          oscNode.type = this.oscType;
        }
        
        oscNode.frequency.setValueAtTime(_.toFinite(freq), ac.currentTime); 

      //BEAT
        var stopTimeBeat = startTime + 0.3;
        var fadeOutTimeBeat = startTime + 0.1
        var gainNodeBeat = ac.createGain();
        gainNodeBeat.gain.setValueAtTime(1, ac.currentTime);
        var oscNodeBeat = ac.createOscillator();
        oscNodeBeat.type = "sine";
        oscNodeBeat.frequency.setValueAtTime(55.00, ac.currentTime);  // C2
        oscNodeBeat.connect(gainNodeBeat);
        if(i % 2 == 0){
          oscNodeBeat.start(ac.currentTime + startTime);
          gainNodeBeat.gain.setTargetAtTime(0, ac.currentTime + fadeOutTimeBeat, 0.01);
          oscNodeBeat.stop(ac.currentTime + stopTimeBeat);
        }
        // END BEAT LOGIC
        
        // Connects
        gainNodeBeat.connect(gainNode);
        oscNode.connect(gainNode);
        
        gainNode.gain.setValueAtTime(0.00001, ac.currentTime + startTime);
        gainNode.connect(this.analyser);
        gainNode.connect(ac.destination);

        //Gain fade in
        gainNode.gain.setTargetAtTime(0.3, ac.currentTime + startTime, 0.01);
        oscNode.start(ac.currentTime + startTime);

        //Gain fade out and stop
        gainNode.gain.setTargetAtTime(0, ac.currentTime + fadeOutTime, 0.02);
        
        //add an eventhandler to the last oscillatorNode as we need close the audio context when it has stopped
        if(i==this.prices.length -1){
          var vm = this;
          oscNode.onended = function() {
            vm.playing = false;
            vm.closeAudioContext(ac);
          };
        }
        oscNode.stop(ac.currentTime + stopTime);
      }
      this.playing = true;
    },
    stopOscillator(){
      this.closeAudioContext();
      this.playing = false
    },
    closeAudioContext(){
      this.ac.close().then(() => {
        console.log("Audio Context ended");
        this.ac = null
      })
      .catch(err => console.error(err));
    },
    getScale(midiArray, scale){
      var newMidiArray = scale.map(index => {
        return midiArray[index]
      })
      return newMidiArray;
    },
    getMidiToFreqArray(){
      let midi = [];
      let exp = null;
      const a = 440; // a is 440 hz...
      for (let x = 0; x <= 127; x++)
      {
        exp =  ((x - 9) / 12);
        midi[x] =  (a / 32) * Math.pow(2, exp);
      }
      return midi;
    },
    changeOscType(type){
      this.oscType = type;
    },
    onWindowsResize(){
      this.w = Math.ceil(window.innerWidth * 0.3);
      this.h = Math.ceil((window.innerWidth/window.innerHeight) * 47);
    }
  },
  
}
</script>

<style scoped>
.main {
  display: flex;
  width:95%;
  margin-top:5%;
  flex-direction: column;
}

h1 {
  margin: 2% 0;
  font-size: 3rem;
  color:#FFF;
  text-align: left;
}

.chart-container {
  padding: 2%;
  border-radius: 10px;
  width: 100% !important;
  align-self: center;
  background: #FFF;
  border: 3px solid #5AAC87;
}

.panel {
  display: flex;
  position:absolute;
  bottom:0;
  width: 50%;
  left:25%;
  padding:1.5% 1% 1% 1%;
  background: #FFF;  
  border-radius: 10px 10px 0 0;
}

.panel button {
  border:0;
  display: block;
  width: 15%;
  height: 0;
  padding-bottom: 15%;
  margin: 0 3% 0 1%;
  outline: 0;
  background:url("../assets/img/playback-controls.svg");  
  background-size:205%;
}

.panel button.play {
  background-position: 0% 103%;
}

.panel button.stop {
  background-position: 94% 103%;
}

.panel .selector {
  display: flex;
  flex-wrap: wrap;
  width: 30%;
  height: 0;
  padding:1% 2% 10% 1%;
}
.panel .selector > div {
  width: 50%;
  padding-bottom: 23%;
  background:url("../assets/img/waves-buttons.svg");
  background-size: 203%;
}
.panel .selector > div.triangle{
  background-position:100% 2%;
}

.panel .selector > div.triangle.active{
  background-position:100% 34%
}

.panel .selector > div.square{
  background-position:-196% 2%;   
}

.panel .selector > div.square.active{
  background-position:-196% 34%;
}

.panel .selector > div.sawtooth{
  background-position:100% 67%;
}

.panel .selector > div.sawtooth.active{
  background-position:100% 99%;
}

.panel .selector > div.sine{
  background-position:-196% 67%;
}

.panel .selector > div.sine.active{
  background-position:-196% 99%;
}

.canvas-wrapper {
  width:50%;
  overflow:hidden;
  background: rgb(163, 215, 224);
}

</style>
