<template>
  <div id="page">
    <vs-row>
      <vs-col vs-w="5">
        <p>id: {{ myId }}</p>
      </vs-col>
      <vs-col vs-w="5">
        <vs-input v-model="remoteId"/>
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
import debounce from 'lodash/debounce'
import Peer from 'skyway-js'

export default {
  name: 'md-editor',
  data() {
    return {
      text: '',
      // SkyWayのpeer
      peer: null,
      // 接続しているpeerを管理するオブジェクト
      connectedPeers: {},
      dataConnection: null,
      myId: '',
      // 接続相手のID
      remoteId: ''
    }
  },
  mounted () {
    // 初期化
    const peer = new Peer({
      key: '',
      debug: 3,
    })

    // シグナリングサーバーに正常に接続できた時
    peer.on('open', () => {
      console.log('peer opened!')
      this.myId = peer.id
    })

    // p2pで接続相手と接続した時
    peer.on('connection', c => {
      console.log('peer connected!')
      this.dataConnection = c
      this.connectedPeers[this.dataConnection.remoteId] = 1
      // データコネクションがデータを受信した時
      c.on('data', data => {
        this.text = data
      })
    })

    // Peerに対する全ての接続を終了した時
    peer.on('close', () => console.log('peer close'))

    peer.on('error', err => console.log(err))
    // peerがmounted内に取り残されてしまうので、dataに入れる
    this.peer = peer
  },
  computed: {
    compiledMarkdown: function () {
      return marked(this.text, { sanitize: true })
    }
  },
  methods: {
    // テキストエリアに変更があった時に走るメソッド
    update: debounce(function (e) {
      this.text = e.target.value
      
      if (this.connectedPeers[this.dataConnection.remoteId]) {
        this.dataConnection.send(this.text)
      }
    }, 50),

    // 接続ボタンが押された時に走るメソッド
    connect: function () {
      const remoteId = this.remoteId
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
