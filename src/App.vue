<template>
    <div id="app">

        <span class="text-xl" v-show="isSelfSpeaking">Speaking...</span>
        <span class="text-xl" v-show="!isSelfSpeaking">Not Speaking</span>
        <lottie :options="defaultOptions" :height="200" :width="200" v-on:animCreated="handleAnimation"/>

        <br>
        <br>

        <div class="flex flex-row">
            <audio class="flex-1" ref="self" id="self" controls></audio>
            <audio class="flex-1" ref="partner" id="partner" controls></audio>
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

        <br>
        <br>

        Recorded chunks: {{ audioChunks.length }}

        <button @click="startRecording" class="border bg-gray p-2 rounded ml-2">Start Recording</button>
        <button @click="stopRecording" class="border bg-gray p-2 rounded ml-2">Stop Recording</button>
        <audio ref="recording" id="recording" controls></audio>

    </div>
</template>

<script>
    import Lottie from 'vue-lottie';
    import Peer from 'peerjs'
    import Hark from 'hark'
    import * as animationData from './assets/sound-bars-animation.json';

    export default {
        name: 'app',

        components: {
            'lottie': Lottie
        },

        data() {
            return {
                host: "23.239.12.93",
                port: 3000,

                self: {},
                partner: {},

                peerId: "",
                invitePeerId: "",
                peer: null,
                mediaStream: null,

                recorder: null,
                audioChunks: [],
                isSelfSpeaking: false,

                defaultOptions: {
                    animationData: animationData.default
                },
                animationSpeed: 1,
                animation: null,
            }
        },

        mounted() {
            this.self = this.$refs.self;
            this.partner = this.$refs.partner;
            this.recording = this.$refs.recording;

            this.getUserVideo()
        },

        methods: {
            connect() {
                this.peer = new Peer(this.peerId, {host: this.host, port: this.port, path: '/api'});
                this.peer.on('call', (call) => {
                    call.answer(this.mediaStream);

                    call.on('stream', stream => {
                        this.partner.srcObject = stream;
                        this.partner.play();
                    });
                })
            },

            invite() {
                const call = this.peer.call(this.invitePeerId, this.mediaStream);
                call.on('stream', stream => {
                    this.partner.srcObject = stream;
                    this.partner.play();
                });
            },

            getUserVideo() {
                navigator.mediaDevices.getUserMedia({audio: true, video: false})
                    .then(stream => {
                        this.mediaStream = stream;
                        this.self.srcObject = this.mediaStream;
                        this.self.muted = true;
                        this.self.play();

                        let options = {};
                        const speechRecognition = Hark(this.mediaStream, options);
                        speechRecognition.on('speaking', () => {
                            this.isSelfSpeaking = true;
                            this.animation.play();
                        });
                        speechRecognition.on('stopped_speaking', () => {
                            this.isSelfSpeaking = false;
                            this.animation.stop();
                        });

                    }).catch(() => {
                        alert("Error! Make sure to click allow when asked for permission by the browser");
                    });
            },

            startRecording() {
                this.audioChunks = [];

                this.recorder = new MediaRecorder(this.mediaStream);
                this.recorder.ondataavailable = event => this.audioChunks.push(event.data);
                this.recorder.onstop = () => this.processAudio();

                this.recorder.start(1000);
            },

            stopRecording() {
                this.recorder.stop();

            },

            processAudio() {
                let audioBlob = new Blob(this.audioChunks);
                this.recording.src = URL.createObjectURL(audioBlob);
            },

            handleAnimation: function(animation) {
                this.animation = animation;
                this.animation.stop();
            },
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
