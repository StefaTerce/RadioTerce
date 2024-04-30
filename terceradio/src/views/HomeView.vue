<template>
 <v-container>
  <v-card
    class="mx-auto"
    max-width="400"
  >
    <v-layout>

      <v-app-bar color="#7fffd4">
        <v-app-bar-nav-icon></v-app-bar-nav-icon>

        <v-toolbar-title>My Music</v-toolbar-title>

        <v-spacer></v-spacer>

        <v-btn icon="mdi-magnify"></v-btn>
      </v-app-bar>

      <v-main>
        <v-container class="containerCards">
          <v-row dense>
            <v-col cols="12">
              <v-card color="#952175">
                <div class="d-flex flex-no-wrap justify-space-between">
                  <div>
                    <v-card-title class="text-h5">
                      Halcyon Days
                    </v-card-title>

                    <v-card-subtitle>Ellie Goulding</v-card-subtitle>

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
                    <v-img src="https://cdn.vuetifyjs.com/images/cards/halcyon.png"></v-img>
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
          this.radios = data;
          console.log(data);
        });
    },
    generateCard()
    {
      //a function that generates cards
      
    },
  },
  created() {
    this.getRadios();
  },
}
</script>
