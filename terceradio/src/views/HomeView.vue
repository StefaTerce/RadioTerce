<template>
  <v-container class="mt-10">
    <v-card class="mx-auto" max-width="1200">
      <v-layout>
        <v-main>
          <v-container class="containerCards">
            <v-row dense>
              <v-col cols="12" md="4" v-for="(radio, index) in radios" :key="index">
                <v-card :color="radio.color">
                  <div class="d-flex flex-no-wrap justify-space-between">
                    <div>
                      <v-card-title class="text-h5">{{ radio.name }}</v-card-title>
                      <v-card-subtitle>{{ radio.artist }}</v-card-subtitle>
                      <v-card-actions>
                        <v-btn class="ms-2" icon :color="radio.isPlaying ? 'red' : ''" :ripple="false" @click="togglePlayback(radio)">
                          <v-icon>{{ radio.isPlaying ? 'mdi-pause' : 'mdi-play' }}</v-icon>
                        </v-btn>
                      </v-card-actions>
                    </div>
                    <v-avatar class="ma-3" rounded="0" size="125">
                      <v-img v-if="radio.imageUrl" :src="radio.imageUrl"></v-img>
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
      audio: new Audio() // Elemento audio per la riproduzione
    }
  },
  methods: {
    getRadios() {
      fetch('https://nl1.api.radio-browser.info/json/stations/search?limit=100&countrycode=IT&hidebroken=true&order=clickcount&reverse=true')
        .then(response => response.json())
        .then(data => {
          console.log(data)
          this.radios = data.map(radio => ({
            name: radio.name,
            artist: radio.artist,
            imageUrl: radio.favicon || null,
            color: '#952175',
            url: radio.url, // URL dell'audio
            stationId: radio.stationuuid, // Aggiungi l'id della stazione
            isPlaying: false // Aggiungi la proprietà per il controllo della riproduzione
          }));
          console.log(this.radios);
        });
    },
    playAudio(url) {
      // Pausa l'audio attualmente in riproduzione (se presente)
      this.audio.pause();
      // Imposta l'URL dell'audio
      this.audio.src = url;
      // Avvia la riproduzione dell'audio
      this.audio.play();
    },
    togglePlayback(radio) {
      if (radio.isPlaying) {
        this.audio.pause(); // Pausa l'audio
      } else {
        this.audio.src = radio.url; // Imposta l'URL dell'audio
        this.audio.play(); // Avvia la riproduzione dell'audio
      }
      radio.isPlaying = !radio.isPlaying; // Inverti lo stato di riproduzione
    }
  },
  created() {
    this.getRadios();
  },
}
</script>
