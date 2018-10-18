<template>
  <div class="photobooth">
    <div class="controls">
      <button @click="takePhoto">Take Photo</button>
      <div class="rgb">
        <label for="rmin">Red Min:</label>
        <input type="range" min=0 max=255 name="rmin">
        <label for="rmax">Red Max:</label>
        <input type="range" min=0 max=255 name="rmax">

        <br>

        <label for="gmin">Green Min:</label>
        <input type="range" min=0 max=255 name="gmin">
        <label for="gmax">Green Max:</label>
        <input type="range" min=0 max=255 name="gmax">

        <br>

        <label for="bmin">Blue Min:</label>
        <input type="range" min=0 max=255 name="bmin">
        <label for="bmax">Blue Max:</label>
        <input type="range" min=0 max=255 name="bmax">
      </div>
    </div>

    <canvas ref="canvas" ></canvas>
    <video ref="video" @canplay="paintToCanvas" ></video>
    <div ref="strip"></div>
    <audio ref="snap" src="http://wesbos.com/demos/photobooth/snap.mp3" hidden></audio>
  </div>
</template>

<script>
export default {
  methods: {
    getVideo () {
      navigator.mediaDevices.getUserMedia({ video: true, audio: false })
        .then(mediaStream => {
          this.$refs.video.srcObject = mediaStream
          this.$refs.video.play()
        })
        .catch(err => {
          console.error('woops', err)
        })
    },

    paintToCanvas () {
      const width = this.$refs.video.videoWidth
      const height = this.$refs.video.videoHeight
      this.$refs.canvas.width = width
      this.$refs.canvas.height = height

      const ctx = this.$refs.canvas.getContext('2d')
      return setInterval(() => {
        ctx.drawImage(this.$refs.video, 0, 0, width, height)
        // let pixels = ctx.getImageData(0, 0, width, height)

        // pixels = this.redEffect(pixels)

        // pixels = this.rgbSplit(pixels)
        // ctx.globalAlpha = 0.8

        // pixels = this.greenScreen(pixels)

        // ctx.putImageData(pixels, 0, 0)
      }, 16)
    },

    takePhoto () {
      this.$refs.snap.currentTime = 0
      this.$refs.snap.play()

      const data = this.$refs.canvas.toDataURL('image/png')
      const link = document.createElement('a')
      link.href = data
      link.setAttribute('download', 'handsome')
      link.textContent = 'Download Image'
      this.$refs.strip.insertBefore(link, this.$refs.strip.firstChild)
    },

    redEffect (pixels) {
      for (let i = 0; i < pixels.data.length; i += 4) {
        pixels.data[i] = pixels.data[i + 0] + 100 // r
        pixels.data[i + 1] = pixels.data[i + 1] - 50 // g
        pixels.data[i + 2] = pixels.data[i + 2] * 0.5 // b
      }
      return pixels
    },

    rgbSplit (pixels) {
      for (let i = 0; i < pixels.data.length; i += 4) {
        pixels.data[i - 150] = pixels.data[i + 0] // r
        pixels.data[i + 500] = pixels.data[i + 1] // g
        pixels.data[i - 550] = pixels.data[i + 2] // b
      }
      return pixels
    },

    greenScreen (pixels) {
      const levels = {}

      document.querySelectorAll('.rgb input').forEach((input) => {
        levels[input.name] = input.value
      })

      for (let i = 0; i < pixels.data.length; i = i + 4) {
        const red = pixels.data[i + 0]
        const green = pixels.data[i + 1]
        const blue = pixels.data[i + 2]
        // const alpha = pixels.data[i + 3]

        if (red >= levels.rmin &&
          green >= levels.gmin &&
          blue >= levels.bmin &&
          red <= levels.rmax &&
          green <= levels.gmax &&
          blue <= levels.bmax) {
          // take it out!
          pixels.data[i + 3] = 0
        }
      }

      return pixels
    }
  },

  mounted () {
    this.getVideo()
  }
}
</script>

<style scoped>
html {
  box-sizing: border-box;
}

*, *:before, *:after {
  box-sizing: inherit;
}

html {
  font-size: 10px;
  background: #ffc600;
}

.photobooth {
  background: white;
  max-width: 150rem;
  margin: 2rem auto;
  border-radius: 2px;
}

/*clearfix*/
.photobooth:after {
  content: '';
  display: block;
  clear: both;
}

canvas {
  width: 100%;
  float: left;
}

video {
  position: absolute;
  top: 20px;
  right: 20px;
  width:200px;
}

/*
  Strip!
*/

.strip {
  padding: 2rem;
}

.strip img {
  width: 100px;
  overflow-x: scroll;
  padding: 0.8rem 0.8rem 2.5rem 0.8rem;
  box-shadow: 0 0 3px rgba(0,0,0,0.2);
  background: white;
}

.strip a:nth-child(5n+1) img { transform: rotate(10deg); }
.strip a:nth-child(5n+2) img { transform: rotate(-2deg); }
.strip a:nth-child(5n+3) img { transform: rotate(8deg); }
.strip a:nth-child(5n+4) img { transform: rotate(-11deg); }
.strip a:nth-child(5n+5) img { transform: rotate(12deg); }

</style>

