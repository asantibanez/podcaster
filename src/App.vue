<template>
    <div id="app">

        <div class="flex flex-row">
            <video class="flex-1" ref="video" id="video" width="640" height="480" autoplay></video>
            <video class="flex-1" ref="partner" id="partnet" width="640" height="480" autoplay></video>
        </div>

        <br>
        <br>
        <input type="text" v-model="host" class="border"/>
        <input type="text" v-model="port" class="border"/>
        <br>
        <br>
        <input type="text" v-model="peerId" class="border"/>
        <button @click="connect" class="border bg-gray p-2 rounded ml-2">Connect</button>
        <br>
        <br>
        <input type="text" v-model="invitePeerId" class="border"/>
        <button @click="invite" class="border bg-gray p-2 rounded ml-2">Invite</button>
    </div>
</template>

<script>
    import Peer from 'peerjs'

    export default {
        name: 'app',

        data() {
            return {
                host: "192.168.100.16",
                port: 3000,

                video: {},
                partner: {},

                peerId: "",
                invitePeerId: "",
                peer: null,
                mediaStream: null,
            }
        },

        mounted() {
            this.video = this.$refs.video;
            this.partner = this.$refs.partner;
            this.getUserVideo()
        },

        methods: {
            connect() {
                this.peer = new Peer(this.peerId, {host: this.host, port: this.port, path: '/api'});
                this.peer.on('call', (call) => {
                    call.answer(this.mediaStream);

                    call.on('stream', stream => {
                        this.partner.srcObject = stream;
                    });
                })
            },

            invite() {
                this.peer.call(this.invitePeerId, this.mediaStream)
            },

            getUserVideo() {
                navigator.getUserMedia = navigator.getUserMedia || navigator.webkitGetUserMedia || navigator.mozGetUserMedia;

                navigator.getUserMedia({audio: false, video: true}, stream => {
                    this.mediaStream = stream;
                    this.video.srcObject = this.mediaStream;
                }, () => {
                    alert("Error! Make sure to click allow when asked for permission by the browser");
                });
            }
        }
    }
</script>

<style>
    #app {
        font-family: 'Avenir', Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
        margin-top: 60px;
    }
</style>
