<template>
  <div class="home">
    <video @loadeddata="onLoadedData" :srcObject.prop="srcObject" playsInline muted></video>
    <div v-for="screen in screens" :key="screen.id">
      <video @loadeddata="onLoadedData" :srcObject.prop="screen" plyasInline muted></video>
    </div>
    <textarea v-model="roomId"></textarea>
    <button @click="join">join</button>
    <h3>{{ logMessage }}</h3>
  </div>
</template>

<script>
import Peer from 'skyway-js';
import { APIKEY } from '../key';

export default {
  name: 'Home',
  data () {
    return {
      peer: null,
      srcObject: null,
      roomId: null,
      localStream: null,
      logMessage: null,
      screens: [],
    }
  },
  async created() {
    this.localStream = await navigator.mediaDevices
      .getUserMedia({
        audio: true,
        video: true,
      })
      .catch(console.error);
    this.srcObject = this.localStream;
    console.log(this.srcObject)

    this.peer = await new Peer({
      key: APIKEY,
      debug: 1,
    })

    console.log(this.peer)

  },
  methods: {
    join() {
      if (!this.peer.open) {
        return;
      }
      const room = this.peer.joinRoom(this.roomId, {
        mode: 'sfu',
        stream: this.localStream,
      })

      room.once('open', () => {
        this.logMessage = 'You joined this room. ===\n';
      })

      room.on('peerJoin', peerId => {
        this.logMessage = this.logMessage + `${peerId} joined ===\n`;
      })

      room.on('stream', async stream => {
        await this.screens.push(stream);
        console.log(this.screens)
      })

      room.on('data', ({ data, src }) => {
        console.log({ data }, { src })
      });
    },
    onLoadedData(event) {
      console.log(event);
      event.target.play().catch(console.error)
    }
  }
}
</script>
