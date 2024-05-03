<template>
  <v-container class="mt-10">
    <v-card class="mx-auto" max-width="1500">
      <v-layout>
        <v-main>
          <v-container class="containerCards">
            <v-row dense>
              <v-col cols="12" md="4" v-for="(radio, index) in FavouriteRadios" :key="index"> <!-- Usa FavouriteRadios qui -->
                <v-card :color="randomColor()">
                  <div class="d-flex flex-no-wrap justify-space-between">
                    <div>
                      <v-card-title class="text-small">{{ radio.name }}</v-card-title>
                      <v-card-subtitle class="text-small">{{ radio.artist }}</v-card-subtitle>
                      <v-card-actions>
                        <v-btn class="ms-2" icon :color="radio.isPlaying ? 'red' : ''" @click="togglePlayback(radio)">
                          <v-icon>{{ radio.isPlaying ? 'mdi-pause' : 'mdi-play' }}</v-icon>
                        </v-btn>
                        <v-btn class="ms-2" icon @click="toggleFavorite(radio)">
                          <v-icon :color="radio.isFavorite ? 'yellow darken-2' : ''">mdi-star</v-icon>
                        </v-btn>
                      </v-card-actions>
                    </div>
                    <v-avatar class="ma-3" rounded="0" size="125">
                      <v-img v-if="radio.isPlaying" :src="require('@/assets/audio-8777_256.gif')"></v-img>
                      <v-img width="100" height="120" v-else-if="radio.imageUrl" :src="radio.imageUrl"></v-img>
                      <v-icon v-else>mdi-radio</v-icon>
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
export default {
  name: 'HomeView',
  data() {
    return {
      radios: [],
      FavouriteRadios: [], // Aggiungi qui l'array per le radio preferite
      audio: new Audio(),
      currentPlayingRadio: null
    };
  },
  methods: {
    randomColor() {
      const colors = ['#952175', '#00796b', '#1976d2', '#c62828'];
      const randomIndex = Math.floor(Math.random() * colors.length);
      return colors[randomIndex];
    },
    getRadios() {
      // Recupera i dati dal localStorage
      const storedRadios = JSON.parse(localStorage.getItem('radios')) || [];
      this.radios = storedRadios; // Salva tutte le radio in 'radios'
      // Filtra le radio preferite e salva in 'FavouriteRadios'
      this.FavouriteRadios = storedRadios.filter(radio => radio.isFavorite);
    },
    playAudio(url, radio) {
      if (this.currentPlayingRadio && this.currentPlayingRadio !== radio) {
        this.currentPlayingRadio.isPlaying = false;
        this.audio.pause();
      }
      this.audio.src = url;
      this.audio.play();
      this.currentPlayingRadio = radio;
    },
    togglePlayback(radio) {
      if (radio.isPlaying) {
        this.audio.pause();
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