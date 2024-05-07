<template>
  <v-container class="mt-10">
    <v-card class="mx-auto" max-width="1500">
      <v-layout>
        <v-main>
          <v-container class="containerCards">
            <v-row dense>
              <v-col cols="12" md="4" v-for="(radio, index) in radios" :key="index">
                <v-card :color="randomColor()">
                  <div class="d-flex flex-column align-center justify-center">
                    <v-avatar class="ma-3" rounded="0" size="125">
                      <v-img v-if="radio.isPlaying" :src="require('@/assets/audio-8777_256.gif')"></v-img>
                      <v-img width="100" height="120" v-else-if="radio.imageUrl" :src="radio.imageUrl"></v-img>
                      <v-icon v-else>mdi-radio</v-icon>
                    </v-avatar>
                    <div class="text-center">
                      <v-card-title class="text-subtitle-2">{{ radio.name }}</v-card-title>
                      <v-card-subtitle class="text-caption">{{ radio.artist }}</v-card-subtitle>
                    </div>
                    <v-card-actions class="justify-center">
                      <v-btn class="ms-2" icon :color="radio.isPlaying ? 'red' : ''" @click="togglePlayback(radio)">
                        <v-icon>{{ radio.isPlaying ? 'mdi-pause' : 'mdi-play' }}</v-icon>
                      </v-btn>
                      <v-btn class="ms-2" icon @click="toggleFavorite(radio)">
                        <v-icon :color="radio.isFavorite ? 'yellow darken-2' : ''">mdi-star</v-icon>
                      </v-btn>
                    </v-card-actions>
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
import Hls from 'hls.js';

export default {
  name: 'HomeView',
  data() {
    return {
      radios: [],
      audio: new Audio(), // Elemento audio per la riproduzione
      currentPlayingRadio: null, // Memorizza l'ID della radio attualmente in riproduzione
      hls: null // Instance di HLS.js
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
            console.log(data);
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
            console.log(this.radios);
          });
      }
    },
    playAudio(url, radio) {
      if (this.currentPlayingRadio && this.currentPlayingRadio !== radio) {
        this.currentPlayingRadio.isPlaying = false;
        if (this.hls) {
          this.hls.destroy();
          this.hls = null;
        }
        this.audio.pause();
      }

      if (url.endsWith('.m3u8')) {
        if (Hls.isSupported()) {
          this.hls = new Hls();
          this.hls.loadSource(url);
          this.hls.attachMedia(this.audio);
          this.hls.on(Hls.Events.MANIFEST_PARSED, () => {
            this.audio.play();
          });
        } else if (this.audio.canPlayType('application/vnd.apple.mpegurl')) {
          this.audio.src = url;
          this.audio.play();
        }
      } else {
        this.audio.src = url;
        this.audio.play();
      }

      this.currentPlayingRadio = radio;
    },
    togglePlayback(radio) {
      if (radio.isPlaying) {
        this.audio.pause();
        if (this.hls) {
          this.hls.stopLoad();
        }
      } else {
        this.playAudio(radio.url, radio);
      }
      radio.isPlaying = !radio.isPlaying;
    },
    toggleFavorite(radio) {
      radio.isFavorite = !radio.isFavorite;
      this.saveToLocalStorage();
    },
    saveToLocalStorage() {
    // Mappa tutte le radio impostando isPlaying a false prima di salvarle
    const radiosToSave = this.radios.map(radio => ({
      ...radio,
      isPlaying: false // Assicura che isPlaying sia sempre false quando salvi nel localStorage
    }));
    localStorage.setItem('radios', JSON.stringify(radiosToSave));
  }
  },
  created() {
    this.getRadios();
  }
};
</script>

<style scoped>
.ma-3 {
  margin-top: 10px; /* Increase top margin if necessary for better visual separation */
}
.text-center {
  text-align: center; /* Ensures the text is centered below the image */
}
</style>

