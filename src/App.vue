<template>
  <div id="app">
    <div class="camera-modal">
      <video id="video" class="camera-stream"/>
    </div>
    <div>
<!--      <button @click="prediction()">Record</button>-->
      <p>{{text}}</p>
    </div>
  </div>
</template>

<script>

const blazeFace = require('@tensorflow-models/blazeface');
import * as tf from '@tensorflow/tfjs-core';
require('@tensorflow/tfjs-backend-webgl');
//import * as tfjsWasm from '@tensorflow/tfjs-backend-webgl';
//import * as dat from 'dat.gui';
export default {
  name: 'App',
  data() {
    return {
      video: {},
      canvas: {},
      captures: [],
      model: {},
      text: '',
      isOn: false,
      timeout: ''
    }
  },
  methods: {
    async cameraUse() {
      const that = this
        that.video = document.getElementById('video');

        const stream = await navigator.mediaDevices.getUserMedia({
          'audio': false,
          'video': {width: 500, height: 500, frameRate: {max: 10}}
        })
      that.video.srcObject = stream;

      //tfjsWasm.setWasmPath('https://cdn.jsdelivr.net/npm/@tensorflow/tfjs-backend-wasm@latest/dist/tfjs-backend-wasm.wasm');

      await tf.setBackend('webgl')
        that.video.play();
      return new Promise((resolve) => {
        that.video.onloadedmetadata = () => {
          resolve(that.video);
        };
      });

    },


    async prediction()  {
      const that = this
      const returnTensors = false;
      const flipHorizontal = true;
      const annotateBoxes = true;
      that.text='';

      const prediction = await that.model.estimateFaces(this.video, returnTensors, flipHorizontal, annotateBoxes);
      //console.log('Idhar hai kya? ')
      console.log('Size of prediction ',prediction.length)
      //prediction.forEach(face => console.log(face.scaledMesh));
      if (prediction.length > 0){
        if(that.timeout > 0){
          console.log('Timeout Reset')
          window.clearTimeout(that.timeout)
          that.timeout = 0;
        }

        console.log('idhar tak aaya');
        for(let i = 0; i < prediction.length; i++){
          //const size = [prediction[i].bottomRight[0] - prediction[i].topLeft[0], prediction[i].bottomRight[1] - prediction[i].topLeft[1]];
          //console.log('Size kya hai ' ,size);
          //console.log(prediction[i]);
          let diffRight = prediction[i].landmarks[0][0] - prediction[i].landmarks[4][0];
          console.log('Right Eye to Ear Distance', diffRight)
          let diffLeft = prediction[i].landmarks[1][0] - prediction[i].landmarks[5][0];
          console.log('Left Eye to Ear Distance', diffLeft)
          const start = prediction[i].topLeft;
          const end = prediction[i].bottomRight;
          let faceDistance = (((start[0]-end[0])/500)*100).toFixed(2);
          console.log('Face Distance ', faceDistance)
          if(diffRight < -15 && diffLeft > 15 && faceDistance >= 20) {
            window.clearTimeout(that.timeout)
            console.log('Timeout Value', that.timeout)
            that.text = 'Hello there',
                that.isOn = true
            console.log(that.text)
          }else{
            that.text = 'Sorry not straight',
                console.log(that.text)
          }
        }
      }
      else{
        that.text = 'Something is wrong in the code'
        console.log('Timeout Initiated')
        console.log(that.text)
        that.timeout = setTimeout(function(){console.log('Timeout ki maki')}, 10000)

      }
      that.text = ''
        requestAnimationFrame(that.prediction);



    },

    async faceDetect() {
      const that = this;
      await this.cameraUse();
      that.video.play();
      that.model = await blazeFace.load();
      await this.prediction();
      // requestAnimationFrame(function(){
      //   setInterval(function () { that.prediction()}, 1000)
      // });
    }

  },
  mounted () {
    this.faceDetect();

  }
}
</script>

<style>
.camera-modal {
  width: 360px;
  height: 360px;
  top: 0;
  left: 0;
  position: absolute;
  background-color: white;
  z-index: 10;
}
.camera-stream {
  width: 100%;
  max-height: 100%;
}

#app {
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
#video {
  background-color: #000000;
}
#canvas {
  display: none;
}
li {
  display: inline;
  padding: 5px;
}
</style>
