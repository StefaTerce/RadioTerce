<template>
  <v-container class="mt-10">
    <v-card class="mx-auto" max-width="2000">
      <v-layout>
        <v-main>
          <v-col cols="12" md="4" class="text-center mt-10">
            <v-select
              v-model="selectedTags"
              :items="allTags"
              label="Seleziona tag"
              multiple
              chips
              dense
            ></v-select>
          </v-col>

          <v-container class="containerCards">
            <v-row dense>
              <v-col cols="12" md="4" v-for="(radio, index) in filteredRadios" :key="index">
                <v-card :color="randomColor()" style="height: 370px;">
                  <div class="d-flex flex-column align-center justify-center">
                    <v-avatar class="ma-3" rounded="0" size="125">
                      <v-img v-if="radio.isPlaying" :src="require('@/assets/audio-8777_256.gif')"></v-img>
                      <v-img width="100" height="120" v-else-if="radio.imageUrl" :src="radio.imageUrl"></v-img>
                      <v-icon v-else>mdi-radio</v-icon>
                    </v-avatar>
                    <div class="text-center">
                      <v-card-title class="text-subtitle-2">{{ radio.name }}</v-card-title>
                      <v-card-subtitle class="text-caption">{{ radio.artist }}</v-card-subtitle>
                      <!-- Tags displayed as chips -->
                      <v-row class="mt-2">
                        <v-chip
                          v-for="tag in radio.tags.split(/[,\s]+/).filter(tag => tag.trim() !== '')"
                          :key="tag"
                          class="ma-1"
                          small
                        >
                          {{ tag }}
                        </v-chip>
                      </v-row>
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
      audio: new Audio(),
      currentPlayingRadio: null,
      hls: null,
      allTags: [],
      selectedTags: []
    };
  },
  computed: {
    filteredRadios() {
      if (this.selectedTags.length === 0) {
        return this.radios;
      }
      return this.radios.filter(radio => {
        const radioTags = radio.tags ? radio.tags.split(/[,\s]+/).map(tag => tag.trim()) : [];
        return this.selectedTags.some(tag => radioTags.includes(tag));
      });
    }
  },
  methods: {
    randomColor() {
      const colors = ['#952175', '#00796b', '#1976d2', '#c62828'];
      return colors[Math.floor(Math.random() * colors.length)];
    },
    creaTag() {
      this.radios.forEach(radio => {
        if (radio.tags) {
          const tagsArray = radio.tags.split(/[,\s]+/).map(tag => tag.trim());
          this.allTags.push(...tagsArray);
        }
      });
      this.allTags = [...new Set(this.allTags)].sort();
    },
    getRadios() {
      const storedRadios = localStorage.getItem('radios');
      if (storedRadios) {
        this.radios = JSON.parse(storedRadios);
        this.creaTag();
      } else {
        fetch('https://nl1.api.radio-browser.info/json/stations/search?limit=100&countrycode=IT&hidebroken=true&order=clickcount&reverse=true')
          .then(response => response.json())
          .then(data => {
            this.radios = data.map(radio => ({
              name: radio.name,
              artist: radio.artist,
              imageUrl: radio.favicon || null,
              tags: radio.tags,
              url: radio.url,
              stationId: radio.stationuuid,
              isPlaying: false,
              isFavorite: false
            }));
            this.creaTag(); });
      }
    },
    playAudio(url, radio) {
      if (this.currentPlayingRadio && this.currentPlayingRadio !== radio) {
        this.currentPlayingRadio.isPlaying = false;
        this.audio.pause();
        if (this.hls) {
          this.hls.destroy();
          this.hls = null;
        }
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
      radio.isPlaying = true;
      this.currentPlayingRadio = radio;
    },
    togglePlayback(radio) {
      if (radio.isPlaying) {
        this.audio.pause();
        if (this.hls) {
          this.hls.stopLoad();
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
      const radiosToSave = this.radios.map(radio => ({ ...radio, isPlaying: false }));
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
  margin-top: 10px;
}

.text-center {
  text-align: center;
}
</style>
