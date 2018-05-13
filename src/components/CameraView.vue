<template>
    <div class="camera-modal">
        <video ref="video" class="camera-stream"/>
        <div class="camera-modal-container">
            <span @click="capture" class="take-picture-button take-picture-button mdl-button mdl-js-button mdl-button--fab mdl-button--colored">
              <i class="material-icons">camera</i>
            </span>
        </div>
    </div>
</template>

<script>
import firebase from '../service/firebase'
import postCat from '../mixins/postCat'

export default {
  mixins: [postCat],
  data() {
    return {
      mediaStream: null
    }
  },
  mounted() {
    navigator.mediaDevices
      .getUserMedia({ audio: false, video: true })
      .then(mediaStream => {
        this.useMedia(mediaStream)
      })
      .catch(error => console.error('getUserMedia() error:', error))
  },
  methods: {
    dataURItoBlob(dataURI) {
      // convert base64 to raw binary data held in a string
      // doesn't handle URLEncoded DataURIs - see SO answer #6850276 for code that does this
      var byteString = atob(dataURI.split(',')[1])

      // separate out the mime component
      var mimeString = dataURI
        .split(',')[0]
        .split(':')[1]
        .split(';')[0]

      // write the bytes of the string to an ArrayBuffer
      var ab = new ArrayBuffer(byteString.length)

      // create a view into the buffer
      var ia = new Uint8Array(ab)

      // set the bytes of the buffer to the correct values
      for (var i = 0; i < byteString.length; i++) {
        ia[i] = byteString.charCodeAt(i)
      }

      // write the ArrayBuffer to a blob, and you're done
      var blob = new Blob([ab], { type: mimeString })
      return blob
    },
    useMedia(mediaStream) {
      this.mediaStream = mediaStream
      this.$refs.video.srcObject = mediaStream
      this.$refs.video.play()
    },
    capture() {
      const mediaStreamTrack = this.mediaStream.getVideoTracks()[0]
      const imageCapture = new window.ImageCapture(mediaStreamTrack)
      var vm = this

      return imageCapture
        .grabFrame()
        .then(bitmap => {
          console.log('bitmap :', bitmap)
          let canvas = document.createElement('canvas')
          let context = canvas.getContext('2d')
          context.drawImage(bitmap, 0, 0, bitmap.width, bitmap.height)
          return canvas.toDataURL()
        })
        .then(bitmap => {
          var blob = vm.dataURItoBlob(bitmap)
          return blob
        })
        .then(blob => {
          firebase.storage
            .ref()
            .child(`images/picture-${new Date().getTime()}`)
            .put(blob)
            .then(res => {
              var path = res.metadata.fullPath
              var storageRef = firebase.storage.ref()
              var url = storageRef.child(path).getDownloadURL()
              return url
            })
            .then(function(url) {
              vm.postCat(url, 'Hello')
            })
        })
    }
  },
  destroyed() {
    const tracks = this.mediaStream.getTracks()
    tracks.map(track => track.stop())
  }
}
</script>

<style scoped>
.camera-modal {
  width: 100%;
  height: 100%;
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
</style>