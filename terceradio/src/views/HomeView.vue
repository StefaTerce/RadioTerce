<template>
  <v-container>
    <v-card
      class="mx-auto"
      max-width="1200"
    >
      <v-layout>

        <v-app-bar color="#7fffd4">

          <v-toolbar-title>Radio Terce</v-toolbar-title>

          <v-spacer></v-spacer>
        </v-app-bar>

        <v-main>
          <v-container class="containerCards">
            <v-row dense>
              <v-col cols="12" md="4" v-for="(radio, index) in radios" :key="index">
                <v-card :color="radio.color">
                  <div class="d-flex flex-no-wrap justify-space-between">
                    <div>
                      <v-card-title class="text-h5">
                        {{ radio.name }}
                      </v-card-title>
                      <v-card-subtitle>{{ radio.artist }}</v-card-subtitle>
                      <v-card-actions>
                        <v-btn
                          class="ms-2"
                          icon="mdi-play"
                          variant="text"
                        ></v-btn>
                      </v-card-actions>
                    </div>
                    <v-avatar
                      class="ma-3"
                      rounded="0"
                      size="125"
                    >
                      <v-img :src="radio.imageUrl"></v-img>
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
    }
  },
  methods: {
    getRadios() {
      fetch('https://nl1.api.radio-browser.info/json/stations/search?limit=100&countrycode=IT&hidebroken=true&order=clickcount&reverse=true')
        .then(response => response.json())
        .then(data => {
          this.radios = data.map(radio => ({
            name: radio.name,
            artist: radio.artist,
            imageUrl: radio.imageUrl,
            color: '#952175' // Imposta il colore della card come preferisci
          }));
          console.log(this.radios);
        });
    },
  },
  created() {
    this.getRadios();
  },
}
</script>
