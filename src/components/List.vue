<template>
  <v-layout fluid >
    <v-flex>
      <v-row id="video-container" style="margin-top: -12px; display: none;">
        <video ref="videoPlayer" class="video-js"></video>
      </v-row>
      <div class="mt-2" v-if="!showVideo">
        <v-icon left color="pink">mdi-heart-pulse</v-icon>
        Trending
      </div>
      <div class="mt-5" v-else>
        <v-btn depressed text small @click="backToMain">
          <v-icon left>mdi-playlist-play</v-icon>
          Main Play List
        </v-btn>
      </div>
      <v-btn  
        large
        absolute
        top
        right 
        color="white"
        icon
        v-if="!showVideo"
        @click="showHash = !showHash">
        <v-icon v-if="showHash">mdi-pound-box-outline</v-icon>
        <v-icon v-else>mdi-pound-box</v-icon>
      </v-btn>

      <v-row class="my-2">
        <v-col
          v-for="(video, index) in items"
          :key="index"
          cols="12"
          sm="6"
          md="6"
          lg="4"
        >
          <v-hover v-slot:default="{ hover }">
            <v-card
              class="mx-auto my-3"
              :elevation="hover ? 15 : 2"
              max-width="400"
              outlined
              v-if="video.payload.inputs.name"
              @click="streamVideo(video.id, video.transaction_hash)"
            >
              <v-img
                class="white--text"
                height="200px"
                style="text-shadow: 0px 2px 2px rgba(0,0,0,1);"
                :src="preview.find(x => x.txnId === video.id).gifSrc && hover ? preview.find(x => x.txnId === video.id).gifSrc : preview.find(x => x.txnId === video.id).imgSrc"
                gradient="to top, rgba(0,0,0,0), rgba(0,0,0,0.5)"
              >
                <v-card-title class="align-end fill-height">
                  <small class="ellipsis">{{video.payload.inputs.name}}</small>
                  <small style="font-size: 12px; right: 10px; bottom: 0px; position: absolute;">
                    {{new Date(video.payload.inputs.duration * 1000).toISOString().substr(11, 8)}}
                  </small>
                 </v-card-title>

                <v-progress-circular
                  v-if="preview.find(x => x.txnId === video.id).loading"
                  style="right: 16px; top: 16px; position: absolute;"
                  indeterminate
                  color="white"
                  size="26"
                ></v-progress-circular>
              </v-img>
              <v-expand-transition>
                <v-card-text v-show="showHash">
                  <div><strong>IPFS Hash: </strong>
                  </div>
                  <span class="text--primary">
                    <small>{{video.payload.inputs._bundleHash}}</small>
                  </span>
                  <div class="mt-1">
                    <strong>Transaction Hash: </strong>
                    <span v-if="video.transaction_hash">
                      <small class="text--primary">{{video.transaction_hash}}</small>
                    </span>
                    <span v-else>
                      N/A
                      <br>
                      <br>
                      <br>
                    </span>
                  </div>
                </v-card-text>
              </v-expand-transition>
            </v-card>
          </v-hover>
        </v-col>
      </v-row>

      <v-btn
        color="primary darken-1"
        dark
        fixed
        bottom
        right
        fab
        v-if="!showVideo"
        @click="dialog = true"
      >
        <v-icon>mdi-plus</v-icon>
      </v-btn>

      <v-dialog v-model="dialog" persistent fullscreen hide-overlay transition="dialog-bottom-transition">
        <v-card>
          <v-toolbar dark top fixed :elevation="0" color="primary">
            <v-btn icon dark @click="dialog = false">
              <v-icon>mdi-arrow-left</v-icon>
            </v-btn>
            <v-toolbar-title>New Video</v-toolbar-title>
            <div class="flex-grow-1"></div>
            <v-toolbar-items>
              <v-btn dark text :disabled="!valid || !address" @click="processVideo()">Upload
              </v-btn>
            </v-toolbar-items>
          </v-toolbar>
          <v-card     
            class="mx-auto"
            dark>
            <v-card-title>
              <v-btn
                @click="show = !show"
                text
                v-if="walletProgress == 100"
              >
                <v-icon left>mdi-wallet</v-icon> 
                <div>Wallet</div>
                <v-icon>{{ show ? 'mdi-chevron-up' : 'mdi-chevron-down' }}</v-icon>
              </v-btn>
              <div v-else> 
                <v-btn text disabled>
                  <v-progress-circular
                    :value="walletProgress"
                    color="light-blue"
                    width="2"
                    size="24"
                    class="mr-2"
                  >
                    <small><small> {{ walletProgress }}</small></small>
                  </v-progress-circular>
                  <small> {{walletStatus}} Wallet</small>
                </v-btn>
              </div>
            </v-card-title>

            <v-card-text>
              <div>Network: {{simba ? simba.metadata.network : 'Checking...'}}</div>
              <div>Address: {{address ? address : 'Checking...'}}</div>
            </v-card-text>

            <v-expand-transition>
              <div v-show="show" class="mx-4 mb-2">
                <small>
                  <div>Seed: <strong>{{seed}}</strong></div>
                </small>
              </div>
            </v-expand-transition>
            <v-divider></v-divider>
          </v-card>

          <v-card class="elevation-0">
            <v-form 
              ref="form" 
              v-model="valid" 
            >
              <v-card-text>
                <v-text-field
                  v-model="name"
                  label="Name"
                  prepend-icon="mdi-card-text-outline"
                  :rules="rules"
                ></v-text-field>

                <v-file-input
                  :rules="rules"
                  accept=".mp4"
                  show-size
                  v-model="videoFile"
                  placeholder="Select your video"
                  prepend-icon="mdi-file-video-outline"
                  label="Mp4 File"
                ></v-file-input>

                <v-row
                  align="center"
                  justify="center"
                >

                <video id="video" v-if="fileURL" autoplay loop muted :src="fileURL" style="max-width: 300px;">
                </video>

                </v-row>
              </v-card-text>
            </v-form>
          </v-card>
        </v-card>

        <v-overlay :value="postingTxn || processingVideo" color="black" opacity="0.8">
          <h1>
            <v-progress-circular class="mr-2 mb-2" indeterminate size="32"></v-progress-circular>
            <span v-if="processingVideo">
              Processing Video...
            </span>
            <span v-else>
              Posting Transaction...
            </span>
          </h1>
        </v-overlay>
      </v-dialog>

      <v-overlay :value="overlay">
        <v-progress-circular indeterminate size="64"></v-progress-circular>
      </v-overlay>
    </v-flex>
  </v-layout>
</template>

<script>
import * as libsimba from '@simbachain/libsimba-js'
import gifshot from 'gifshot'
import videojs from 'video.js';

  export default {
    computed:{
      combined(){
        return this.img && this.gif
      }
    },
    watch: {
      videoFile: {
        handler(){
          if (this.videoFile) {
            this.fileURL = URL.createObjectURL(this.videoFile) 
          }
        }
      },
      combined(value){
        if (value) {
          this.constructData()
        }
      },
      dialog: {
        handler(){
          if(!this.dialog) {
            this.resetForm()
          }
        }
      }
    },
    data: () => ({
      loadingVideo: false,
      videoFile: null,
      fileURL: null,
      showHash: false,
      showVideo: false,
      overlay: true,
      polling: null,
      img: null,
      gif: null,
      rules: [v => !!v || 'Field is required'],
      dialog: false,
      show: false,
      name: '',
      wallet: null,
      address: null,
      seed: null,
      walletProgress: 0,
      walletStatus: 'Checking ',
      simba: null,
      postingTxn: false,
      processingVideo: false,
      valid: false,
      items: [],
      preview: [],
      videoOptions: {
        autoplay: true,
        controls: true,
        aspectRatio: "16:9",
        nativeControlsForTouch: true,
        fluid: true,
        sources: [
          {
            src: "/path/to/video.mp4",
            type: "video/mp4"
          }
        ]
      },
      player: null
    }),
    mounted () {
      this.initInstance()
    },
    beforeDestroy () {
      clearInterval(this.polling)
      if (this.player) {
        this.player.dispose()
      }
    },
    created () {
      this.pollData()
    },

    methods: {
      pollData () {
        this.polling = setInterval(() => {
          this.getTxns()
        }, 300000)
      },

      async initInstance() {
        let simba = await libsimba.getSimbaInstance(
          'yourApiUrl',
          null,
          'yourApiKey');
        this.simba = simba;
        this.initWallet();
        this.getTxns();
      },

      async initWallet() {
        let wallet = new libsimba.LocalWallet(
          () => { return Promise.resolve(true) }
        );
        this.wallet = wallet;

        if (wallet.walletExists()) {
          this.walletStatus = 'Unlocking ';
          let lastProgress = 0;

          await wallet.unlockWallet(
            'password',
            (progress) => {
              if (Math.floor(progress * 10) > lastProgress) {
                lastProgress = progress * 10;
                this.walletProgress = Math.floor(progress*100)
              }
            }
          );
          this.simba.setWallet(this.wallet);
          this.address = await wallet.getAddress();
          this.seed = await wallet.getMnemonic();
          return;
        } 

        this.walletStatus = 'Creating ';
        let lastProgress = 0;

        await wallet.generateWallet(
          'password',
          (progress) => {
            if (Math.floor(progress * 10) > lastProgress) {
              lastProgress = progress * 10;
              this.walletProgress = Math.floor(progress*100)
            }
          });
        this.simba.setWallet(this.wallet);
        this.address = await wallet.getAddress();
        this.seed = await wallet.getMnemonic();
      },

      async getTxns() {
        if (!this.simba) {
          return
        }
        let response = await this.simba.getTransactions({});

        this.items = response.data();
        this.overlay = false;
        this.initPreview();
      },

      initPreview() {
        let self = this;
        this.items.forEach(function(item) {
          if (!self.preview.find(x => x.txnId === item.id)) {
            self.preview.push(
              {txnId: item.id, 
                bundleHash: item.payload.inputs._bundleHash, 
                imgSrc: 'https://source.unsplash.com/ygUPa_SaXKw/1600x900',
                gifSrc: '',
                loading: true
              });
            self.getSnapshot(item.id)
          }
        });
      },

      async getSnapshot(txnId) {
        let blob = await this.simba.getFileFromBundleForTransaction(txnId, 1, false);

        let self = this;
        let reader = new FileReader();
        reader.readAsDataURL(blob);
        reader.onload = function () {
          self.preview.find(x => x.txnId === txnId).imgSrc = reader.result
        };
        this.getGif(txnId);
        this.setLoading(txnId, false);
      },

      async getGif(txnId) {
          let blob = await this.simba.getFileFromBundleForTransaction(txnId, 2, false);
          let self = this;
          let reader = new FileReader();
          reader.readAsDataURL(blob);
          reader.onload = function () {
            self.preview.find(x => x.txnId === txnId).gifSrc = reader.result
          };
          this.setLoading(txnId, false);
      },

      processVideo() {
        this.processingVideo = true;
        this.takeSnapshot();
        this.makeGif();
      },

      takeSnapshot() {
        let self = this
        gifshot.takeSnapShot({
          'video': [this.fileURL],
          'gifWidth': 1200,
          'gifHeight': 800,
        }, function (obj) {
          if (!obj.error) {
            self.urltoFile(obj.image, 'snapshot.jpg')
            .then(function(file){
              self.img = file
            })
          } else {
            self.processingVideo = false
          }
        })
      },

      makeGif() {
        let self = this;
        gifshot.createGIF({
          'video': [this.fileURL],
          'interval': 0.1,
          'numFrames': 30,
          'frameDuration': 2,
          'gifWidth': 300,
          'gifHeight': 200,
        },function(obj) {
          if (!obj.error) {
            self.urltoFile(obj.image, 'animated.gif')
            .then(function(file){
              self.gif = file
            })
          } else {
            self.processingVideo = false
          }
        });
      },

      urltoFile(url, filename, mimeType){
        mimeType = mimeType || (url.match(/^data:([^;]+);/)||'')[1];
        return (fetch(url)
          .then(function(res){return res.arrayBuffer();})
          .then(function(buf){return new File([buf], filename, {type:mimeType});})
        );
      },

      constructData() {
        let methodParams = { name: this.name, duration: document.getElementById("video").duration };
        let files = [this.videoFile, this.img, this.gif];
        this.postTxn(methodParams, files);
      },

      async postTxn(methodParams, files) {
        this.processingVideo = false
        this.postingTxn = true

        let ret = await this.simba.callMethodWithFile('processVideo', methodParams, files);
        this.dialog = false;
        this.postingTxn = false;
        this.getTxns();
      },

      async streamVideo(txnId, bundleHash) {
        if (this.loadingVideo) {
          return
        } 
        this.loadingVideo = true
        this.setLoading(txnId, true);
        let blob = await this.simba.getFileFromBundleForTransaction(txnId, 0, false);
        this.loadingVideo = false
        this.showVideo = true
        this.videoOptions.sources = [
          {
            src: URL.createObjectURL(blob),
            type: 'video/mp4'
          }
        ];

        if (this.player) {
          this.player.src(this.videoOptions.sources)
          this.player.load()
          this.player.play()
          document.getElementById("video-container").style.display = "block";
          this.setLoading(txnId, false);
          this.scrollToVideo()
          return
        }

        let self = this
        setImmediate(()=>{
          this.player = videojs(this.$refs.videoPlayer, this.videoOptions, function onPlayerReady() {
            document.getElementById("video-container").style.display = "block";
            self.setLoading(txnId, false);
            self.scrollToVideo()
          });
        });
      },

      setLoading(txnId, value) {
        this.preview.find(x => x.txnId === txnId).loading = value
      },

      scrollToVideo() {
        document.getElementById('video-container').scrollIntoView({ behavior: 'smooth', block: 'center'})
      },

      backToMain() {
        if (this.player) {
          this.player.pause()
        }
        document.getElementById("video-container").style.display = "none";
        this.showVideo = false
        this.loadingVideo = false
      },

      resetForm () {
        this.$refs.form.reset();
        this.img = null;
        this.gif = null;
        this.fileURL = null;
        this.videoFile = null
      },
    }
  }
</script>

<style>
.ellipsis {
  width: 320px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
</style>