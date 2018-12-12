<template>
  <div id="editor">
    <vs-row>
      <p>id: {{ myId }}</p>
    </vs-row>
    <vs-row>
      <vs-col vs-type="flex" vs-justify="center" vs-w="6">
        <textarea v-model="input" />
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
      input: '',
      peer: null,
      connectedPeers: null,
      myId: '',

    }
  },
  mounted () {
    this.peer = new Peer({
      key: '',
      debug: 3,
    })

    this.peer.on('open', () => {
      this.id = peer.id
    })

    this.peer.on('connection', c => {
      c.on('open', () => this.connect(c));
    });

    this.peer.on('error', err => console.log(err))
  },
  computed: {
    compiledMarkdown: function () {
      return marked(this.input, { sanitize: true })
    }
  },
  methods: {
    update: _.debounce(function (e) {
      this.input = e.target.value
      const requestedPeer = $('#rid').val();
      if (!this.connectedPeers[requestedPeer]) {
        // Create 2 connections, one labelled chat and another labelled file.
        const c = this.peer.connect(requestedPeer, {
          label:    'chat',
          metadata: {message: 'hi i want to chat with you!'},
        });

        c.on('open', () => {
          this.connect(c);
          this.connectedPeers[requestedPeer] = 1;
        });

        c.on('error', err => alert(err));

        const f = this.peer.connect(requestedPeer, {label: 'file', reliable: true});

        f.on('open', () => {
          this.connect(f);
        });

        f.on('error', err => alert(err));
      }
    }, 300),

    connect: function (c) {


    }

  }
}
</script>
<style scoped>
  #editor, .vs-row {
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
