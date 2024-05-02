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
                      <v-card-title class="text-h5">{{ radio.name }}</v-card-title>
                      <v-card-subtitle>{{ radio.artist }}</v-card-subtitle>
                      <v-card-actions>
                        <v-btn class="ms-2" icon :color="radio.isPlaying ? 'red' : ''" :ripple="false" @click="togglePlayback(radio)">
                          <v-icon>{{ radio.isPlaying ? 'mdi-pause' : 'mdi-play' }}</v-icon>
                        </v-btn>
                        <!-- Aggiungi il tasto preferito -->
                        <v-btn class="ms-2" icon @click="toggleFavorite(radio)">
                          <v-icon :color="radio.isFavorite ? 'yellow darken-2' : ''">mdi-star</v-icon>
                        </v-btn>
                      </v-card-actions>
                    </div>
                    <v-avatar class="ma-3" rounded="0" size="125">
                      <v-img width="100" height="120" v-if="radio.imageUrl" :src="radio.imageUrl"></v-img>
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
      audio: new Audio(), // Elemento audio per la riproduzione
      currentPlayingRadio: null // Memorizza l'ID della radio attualmente in riproduzione
    };
  },
  methods: {
    randomColor() {
      const colors = ['#952175', '#00796b', '#1976d2', '#c62828']; // Array di colori
      const randomIndex = Math.floor(Math.random() * colors.length); // Seleziona un indice casuale nell'array
      return colors[randomIndex]; // Restituisce il colore corrispondente all'indice casuale
    },
    getRadios() {
      fetch('https://nl1.api.radio-browser.info/json/stations/search?limit=100&countrycode=IT&hidebroken=true&order=clickcount&reverse=true')
        .then(response => response.json())
        .then(data => {
          console.log(data);
          this.radios = data.map(radio => ({
            name: radio.name,
            artist: radio.artist,
            imageUrl: radio.favicon || null,
            color: this.randomColor(), // Genera un colore casuale
            url: radio.url, // URL dell'audio
            stationId: radio.stationuuid, // Aggiungi l'id della stazione
            isPlaying: false, // Aggiungi la proprietà per il controllo della riproduzione
            isFavorite: false // Aggiungi la proprietà per indicare se la radio è preferita o meno
          }));
          console.log(this.radios);
        });
    },
    playAudio(url, radio) {
      // Pausa l'audio attualmente in riproduzione (se presente)
      if (this.currentPlayingRadio && this.currentPlayingRadio !== radio) {
        this.currentPlayingRadio.isPlaying = false; // Ferma la riproduzione della radio attualmente in riproduzione
        this.audio.pause();
      }
      // Imposta l'URL dell'audio
      this.audio.src = url;
      // Avvia la riproduzione dell'audio
      this.audio.play();
      // Memorizza l'ID della radio attualmente in riproduzione
      this.currentPlayingRadio = radio;
    },
    togglePlayback(radio) {
      if (radio.isPlaying) {
        this.audio.pause(); // Pausa l'audio
      } else {
        this.playAudio(radio.url, radio); // Avvia la riproduzione dell'audio per questa radio
      }
      // Inverti lo stato di riproduzione
      radio.isPlaying = !radio.isPlaying;
    },
    toggleFavorite(radio) {
      // Inverti lo stato di preferenza della radio
      radio.isFavorite = !radio.isFavorite;
      // Salva i dati aggiornati nel local storage
      this.saveToLocalStorage();
    },
    saveToLocalStorage() {
      // Converti l'array di radio in formato JSON e salvalo nel local storage
      localStorage.setItem('radios', JSON.stringify(this.radios));
    }
  },
  created() {
    this.getRadios();
  }
};
</script>
