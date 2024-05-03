<template>
  <v-container class="mt-10">
    <v-card class="mx-auto" max-width="1500">
      <v-layout>
        <v-main>
          <v-container class="containerCards">
            <v-row dense>
              <v-col cols="12" md="4" v-for="(radio, index) in radios" :key="index">
                <v-card :color="randomColor()">
                  <div class="d-flex flex-no-wrap justify-space-between">
                    <div>
                      <v-card-title class="text-subtitle-2">{{ radio.name }}</v-card-title>
                      <v-card-subtitle class="text-caption">{{ radio.artist }}</v-card-subtitle>
                      <v-card-actions>
                        <v-btn class="ms-2" icon :color="radio.isPlaying ? 'red' : ''" :ripple="false" @click="togglePlayback(radio)">
                          <v-icon>{{ radio.isPlaying ? 'mdi-pause' : 'mdi-play' }}</v-icon>
                        </v-btn>
                        <v-btn class="ms-2" icon @click="toggleFavorite(radio)">
                          <v-icon :color="radio.isFavorite ? 'yellow darken-2' : ''">mdi-star</v-icon>
                        </v-btn>
                      </v-card-actions>
                    </div>
                    <v-avatar class="ma-3" rounded="0" size="125">
                      <v-img width="100" height="120" v-if="radio.imageUrl" :src="radio.imageUrl"></v-img>
                      <v-icon v-else>mdi-radio</v-icon>
                    <!-- Template -->
                    <VideoPlayer
                      v-if="radio.url.endsWith('.m3u8') && radio.isPlaying"
                      ref="hlsPlayer"
                      :link="radio.url"
                      :isMuted="true"
                      :isControls="true"
                      class="video-hls"
                    />

                    </v-avatar>
                  </div>
                </v-card>
              </v-col>
            </v-row>
          </v-container>
        </v-main>
      </v-layout>
    </v-card>
  </v-container>
</template>


<script>
import { VideoPlayer } from 'vue-hls-video-player';

export default {
  name: 'HomeView',
  components: {
    VideoPlayer
  },
  data() {
    return {
      radios: [],
      audio: new Audio(),
      currentPlayingRadio: null,
    };
  },
  methods: {
    randomColor() {
      const colors = ['#952175', '#00796b', '#1976d2', '#c62828'];
      const randomIndex = Math.floor(Math.random() * colors.length);
      return colors[randomIndex];
    },
    getRadios() {
      const storedRadios = localStorage.getItem('radios');
      if (storedRadios) {
        this.radios = JSON.parse(storedRadios);
      } else {
        fetch('https://nl1.api.radio-browser.info/json/stations/search?limit=100&countrycode=IT&hidebroken=true&order=clickcount&reverse=true')
          .then(response => response.json())
          .then(data => {
            this.radios = data.map(radio => ({
              name: radio.name,
              artist: radio.artist,
              imageUrl: radio.favicon || null,
              color: this.randomColor(),
              url: radio.url,
              stationId: radio.stationuuid,
              isPlaying: false,
              isFavorite: false
            }));
            this.$nextTick(() => {
              this.radios.forEach(radio => {
                if (radio.url.endsWith('.m3u8')) {
                  // Pre-initialize or check player readiness here if needed
                }
              });
            });
          });
      }
    },
    playAudio(url, radio) {
      if (url.endsWith('.m3u8')) {
        this.$nextTick(() => {
          if (this.$refs.hlsPlayer) {
            this.$refs.hlsPlayer.link = url;
            this.$refs.hlsPlayer.play();
          } else {
            console.error("HLS Player not available or not properly initialized.");
          }
        });
      } else {
        if (this.currentPlayingRadio && this.currentPlayingRadio !== radio) {
          this.currentPlayingRadio.isPlaying = false;
          this.audio.pause();
        }
        this.audio.src = url;
        this.audio.play();
        this.currentPlayingRadio = radio;
      }
      radio.isPlaying = true;
    },
    togglePlayback(radio) {
      if (radio.isPlaying) {
        if (radio.url.endsWith('.m3u8') && this.$refs.hlsPlayer) {
          this.$refs.hlsPlayer.pause();
        } else {
          this.audio.pause();
        }
        radio.isPlaying = false;
      } else {
        this.playAudio(radio.url, radio);
      }
    },
    toggleFavorite(radio) {
      radio.isFavorite = !radio.isFavorite;
      this.saveToLocalStorage();
    },
    saveToLocalStorage() {
      localStorage.setItem('radios', JSON.stringify(this.radios));
    }
  },
  created() {
    this.getRadios();
  }
};
</script>

<style scoped>
.text-small {
  font-size: 0.875rem; /* Regola questa dimensione come preferisci */
}
</style>
