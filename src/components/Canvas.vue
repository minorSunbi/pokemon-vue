<template>

  <div class="mode-tool column q-mx-xs">
      <q-btn round :color="mode == 'erase' ? 'primary' : 'info'" icon="ti-eraser" @click="eraseMode" />
      <q-btn round :color="mode == 'brush' ? 'primary' : 'info'" icon="ti-pencil" @click="brushMode" />
  </div>
  <div class="vert">
    <div class="canvas" id="input-container"></div>
    
    <div class="modify-tool q-gutter-xs q-my-md q-px-auto">
      <q-btn color="accent" icon="ti-close" label="모두 지우기" @click="clear" />
      <q-btn color="accent" icon="ti-back-left" label="되돌리기" @click="undo" /> 
    </div>
  </div>
  
  <div>
    <q-btn color="accent" icon="ti-control-play" @click="transfer" />    
  </div>

  <q-img class="canvas" :src="outputUrl" crossorigin="anonymous">
    <template v-slot:loading>
      <q-spinner-gears
        color="red"
        size="3rem"
        :thickness="5"
      />
    </template>
  </q-img>
</template>

<script>
const pix2pix = require('../utils/pix2pix.js')
// require('../utils/process.js')
let model;
let p5Canvas;

export default {
  data() {
    return {
      outputUrl: "images/output.png",
      isLoaded: false,
      isTransfering: false,
      mode: 'brush',
      p5Canvas: null,
    }
  },
  methods: {
    eraseMode() {
      this.mode = 'erase'
      p5Canvas.stroke(255);
      p5Canvas.strokeWidth(15);
    },
    brushMode() {
      this.mode = 'brush'
      p5Canvas.stroke(0);
      p5Canvas.strokeWeight(1);
    },
    clear() {
      p5Canvas.background(255)
    },
    undo() {
      // later
    },
    transfer() {
      this.isTransfering = true
      const canvasElem = document.getElementsByTagName('canvas')[0]
      // console.log(canvasElem)
      model.transfer(canvasElem, result => {
        console.log(result)
        this.outputUrl = result.src
        this.isTransfering = false
      })
    }
  },
  computed: {
    
  },
  created() {
    // load pix2pix model
    model = pix2pix('models/pokemonModel.pict', () => {
      // this.model = model
      this.isLoaded = true
      console.log('model loaded')
    })
  },
  mounted() {
    const script = p5 => {
      p5.setup = () => {
        const inputCanvas = p5.createCanvas(256, 256)
      
        p5.loadImage('images/input.png', inputImg => {
          p5.image(inputImg, 0, 0)
        })

        p5.stroke(0)
        p5.pixelDensity(1)
      }

      p5.draw = () => {
        if (p5.mouseIsPressed) {
          p5.line(p5.mouseX, p5.mouseY, p5.pmouseX, p5.pmouseY);
        }
      }

      p5.mouseReleased = () => {
        if (this.isLoaded && !this.isTransfering) {
          this.transfer()
        }
      }

      // p5.eraseMode = () => {
      //   p5.stroke(255);
      //   p5.strokeWidth(15);
      // }
    }

    p5Canvas = new p5(script, 'input-container')
  },
}
</script>

<style>
.canvas {
  box-sizing: content-box;
  width: 256px;
  height: 256px;
  border: 5px solid black;
}
</style>