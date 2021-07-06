<template>
  <div class="container">
    <div class="tool q-my-md q-gutter-xs">
        <q-btn round :color="mode == 'erase' ? 'primary' : 'info'" icon="ti-eraser" @click="eraseMode" />
        <q-btn round :color="mode == 'brush' ? 'primary' : 'info'" icon="ti-pencil" @click="brushMode" />
    </div>

    <div class="flex flex-center q-gutter-md">
      <div class="canvas" id="input-container"></div>

      <q-btn color="accent" icon="ti-control-play" @click="transfer" />    

      <q-img class="canvas" :src="outputUrl" crossorigin="anonymous">
        <template v-slot:loading>
          <q-spinner-gears
            color="red"
            size="3rem"
            :thickness="5"
          />
        </template>
      </q-img>
    </div>

    <div class="q-my-md">
      <q-btn color="accent" icon="ti-close" label="모두 지우기" @click="clear" />
    </div>
  </div>
</template>

<script>
const pix2pix = require('../utils/pix2pix.js')
let model;
let p5Canvas;
let px, py;

export default {
  data() {
    return {
      outputUrl: "images/output.png",
      isLoaded: false,
      isTransfering: false,
      mode: 'brush',
    }
  },
  methods: {
    eraseMode() {
      this.mode = 'erase'
      p5Canvas.stroke(255);
      p5Canvas.strokeWeight(15);
    },
    brushMode() {
      this.mode = 'brush'
      p5Canvas.stroke(0);
      p5Canvas.strokeWeight(1);
    },
    clear() {
      p5Canvas.background(255)
      this.transfer()
    },
    transfer() {
      this.$q.loading.show()
      this.isTransfering = true
      const canvasElem = document.getElementsByTagName('canvas')[0]
      model.transfer(canvasElem, result => {
        this.outputUrl = result.src
        this.isTransfering = false
        this.$q.loading.hide()
      })
    }
  },
  created() {
    // load pix2pix model
    model = pix2pix('models/pokemonModel.pict', () => {
      this.isLoaded = true
      console.log('model loaded')
    })
    this.$q.loading.show()
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
        p5.mouseX = p5.mouseY = undefined
        if (this.isLoaded && !this.isTransfering) {
          this.transfer()
        }
      }
    }

    p5Canvas = new p5(script, 'input-container')
    this.$q.loading.hide()
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