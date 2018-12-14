<template>
  <div id="page">
    <vs-row>
      <vs-col vs-w="5">
        <p>id: {{ myId }}</p>
      </vs-col>
      <vs-col vs-w="5">
        <vs-input v-model="requestedPeerId"/>
      </vs-col>
      <vs-col vs-w="2">
        <vs-button color="primary" @click="connect">接続</vs-button>
      </vs-col>
    </vs-row>
    <vs-row class="editor">
      <vs-col vs-type="flex" vs-justify="center" vs-w="6">
        <textarea v-model="text" @input="update"/>
      </vs-col>
      <vs-col class="previewArea" vs-type="flex" vs-w="6">
        <div v-html="compiledMarkdown" ></div>
      </vs-col>
    </vs-row>
  </div>

</template>

<script>
import marked from 'marked'
import _ from 'lodash'
import Peer from 'skyway-js';

export default {
  name: 'md-editor',
  data() {
    return {
      text: '',
      peer: null,
      connectedPeers: {},
      dataConnection: null,
      myId: '',
      requestedPeerId: ''
    }
  },
  mounted () {
    const peer = new Peer({
    this.peer = new Peer({
      key: '',
      debug: 3,
    })

    peer.on('open', () => {
      console.log('peer opened!')
      this.myId = this.peer.id
    })

    peer.on('connection', c => {
      console.log('peer connected!')
      this.dataConnection = c
      this.connectedPeers[this.dataConnection.remoteId] = 1
      c.on('data', data => {
        this.text = data
      })
    })

    peer.on('close', () => {

    })

    peer.on('error', err => console.log(err))
    this.peer = peer
  },
  computed: {
    compiledMarkdown: function () {
      return marked(this.text, { sanitize: true })
    }
  },
  methods: {
    update: _.debounce(function (e) {
      this.text = e.target.value
      console.log('this.text', this.text)
      if (this.connectedPeers[this.dataConnection.remoteId]) {
        this.dataConnection.send(this.text)
      }
    }, 300),

    connect: function () {
      const remoteId = this.requestedPeerId
      if (!this.connectedPeers[remoteId]) {
        this.dataConnection = this.peer.connect(remoteId)
        this.dataConnection.on('data', data => {
          this.text = data
        })

        this.connectedPeers[this.dataConnection.remoteId] = 1
      }
    }
  }
}
</script>
<style scoped>
  #page {
    height: 100%;
  }

  .editor {
    height: 100%;
  }

  textarea {
    height: 100%;
    width: 100%;
    line-height: 1.5em;
    font-size: 1em;
  }

  .previewArea {
    background-color: beige;
  }
</style>
