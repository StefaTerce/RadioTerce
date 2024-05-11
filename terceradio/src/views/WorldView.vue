<template>
    <div ref="container" class="container"></div>
    <v-dialog v-model="showModal" persistent max-width="600px">
      <v-card>
        <!-- Use a clickable image to represent the station -->
        <template v-if="selectedStation.isPlaying">
          <v-img :src="require('@/assets/audio-8777_256.gif')" height="200px" @click="redirectToSite(selectedStation)"></v-img>
        </template>
        <template v-else-if="selectedStation.imageUrl">
          <v-img width="130" height="120" :src="selectedStation.imageUrl" @click="redirectToSite(selectedStation)"></v-img>
        </template>
        <template v-else-if="selectedStation.favicon">
          <v-img width="130" height="120" :src="selectedStation.favicon" @click="redirectToSite(selectedStation)"></v-img>
        </template>
        <template v-else>
          <v-icon @click="redirectToSite(selectedStation)">mdi-radio</v-icon> <!-- Use mdi-radio as default icon -->
        </template>
        <v-card-title>
          {{ selectedStation ? selectedStation.name : 'Station Details' }}
        </v-card-title>
        <v-card-text>
          <div v-if="selectedStation">
            <p>URL: {{ selectedStation.url }}</p>
            <p>Country: {{ selectedStation.country }}</p>
            <p v-if="selectedStation.site">
              Site Link: <a class="site-link" :href="selectedStation.site" target="_blank">{{ selectedStation.site }}</a>
            </p>
            <!-- Add more details as needed -->
            <v-btn color="primary" @click="togglePlayback(selectedStation)">
              {{ selectedStation.isPlaying ? 'Pause' : 'Play' }}
            </v-btn>
          </div>
          <div v-else>
            <p>No station selected.</p>
          </div>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="stopAudioAndCloseModal">Close</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </template>
  
  <script>
  import * as THREE from 'three';
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
  import Hls from 'hls.js'; // Import HLS library
  import earthTexture from '../assets/8k_earth_daymap.jpg';
  
  export default {
    name: 'ThreeJsScene',
    data() {
      return {
        camera: null,
        renderer: null,
        controls: null,
        earthRadius: 1,
        showModal: false, // Control the visibility of the modal
        selectedStation: null, // Holds the selected radio station's data
        audioPlayer: new Audio(), // Audio player for .mp3 files
        hls: null, // HLS instance
        videoPlayer: null, // Video player for .m3u8 streams
        radioStations: [] // Array to store radio stations data
      };
    },
    mounted() {
      this.init();
      this.animate();
  
      this.renderer.domElement.addEventListener('click', this.onMarkerClick, false);
  
      // Fetch radio stations data for the whole world
      this.fetchRadioStations('https://nl1.api.radio-browser.info/json/stations/search?limit=100&hidebroken=true&order=clickcount&reverse=true&haslocation=true');
  
      // Fetch radio stations data for Italy
      this.fetchRadioStations('https://nl1.api.radio-browser.info/json/stations/search?limit=200&hidebroken=true&order=clickcount&reverse=true&country=Italy');
  
      window.addEventListener('resize', this.handleWindowResize);
  
      // Initialize the video player
      this.videoPlayer = document.createElement('video');
      this.videoPlayer.style.display = 'none'; // Hide the video player
      document.body.appendChild(this.videoPlayer); // Append the video player to the body
    },
  
    beforeUnmount() {
      window.removeEventListener('resize', this.handleWindowResize);
      // Clean up the video player before unmounting the component
      if (this.videoPlayer) {
        this.videoPlayer.pause();
        document.body.removeChild(this.videoPlayer);
      }
    },
    methods: {
      init() {
        this.camera = new THREE.PerspectiveCamera(
          75,
          window.innerWidth / window.innerHeight,
          0.1,
          1000
        );
  
        // Manual computation to roughly position the camera towards Italy
        const phi = (90 - 42.5) * (Math.PI / 180);
        const theta = (12.5 + 180) * (Math.PI / 180);
        const distance = 1.5; // set a suitable distance that best fits your view needs
        this.camera.position.x =
          -distance * Math.sin(phi) * Math.cos(theta);
        this.camera.position.y = distance * Math.cos(phi);
        this.camera.position.z =
          distance * Math.sin(phi) * Math.sin(theta);
  
        this.camera.lookAt(0, 0, 0); // Ensure the camera looks at the center of the globe
  
        this.renderer = new THREE.WebGLRenderer();
        this.renderer.setSize(window.innerWidth, window.innerHeight);
        this.$refs.container.appendChild(this.renderer.domElement);
  
        this.controls = new OrbitControls(
          this.camera,
          this.renderer.domElement
        );
        this.controls.enableDamping = true;
  
        const scene = new THREE.Scene();
  
        const geometry = new THREE.SphereGeometry(this.earthRadius, 64, 64);
        const texture = new THREE.TextureLoader().load(earthTexture);
        const material = new THREE.MeshPhongMaterial({ map: texture });
        const earth = new THREE.Mesh(geometry, material);
        scene.add(earth);
  
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
        scene.add(ambientLight);
  
        const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
        directionalLight.position.set(1, 1, 1).normalize();
        scene.add(directionalLight);
  
        this.scene = scene;
      },
      onMarkerClick(event) {
        event.preventDefault();
  
        const mouse = new THREE.Vector2();
        mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
        mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
  
        const raycaster = new THREE.Raycaster();
        raycaster.setFromCamera(mouse, this.camera);
  
        const intersects = raycaster.intersectObjects(this.scene.children);
  
        if (intersects.length > 0) {
          const intersected = intersects[0].object;
          if (intersected.userData.name) {
            console.log('Clicked on radio station:', intersected.userData);
            this.selectedStation = intersected.userData;
            this.showModal = true;
          }
        }
      },
  
      fetchRadioStations(url) {
        fetch(url)
          .then((response) => response.json())
          .then((data) => {
            this.radioStations = this.radioStations.concat(data); // Store radio stations data
            this.addMarkers(); // Add markers for each radio station
          })
          .catch((error) => {
            console.error('Error fetching radio station data:', error);
          });
      },
  
      addMarkers() {
        this.radioStations.forEach((station) => {
          if (station.geo_long !== null && station.geo_lat !== null) {
            const phi = (90 - station.geo_lat) * (Math.PI / 180);
            const theta = (station.geo_long + 180) * (Math.PI / 180);
            const x = -this.earthRadius * Math.sin(phi) * Math.cos(theta);
            const y = this.earthRadius * Math.cos(phi);
            const z = this.earthRadius * Math.sin(phi) * Math.sin(theta);
  
            const geometry = new THREE.SphereGeometry(0.025, 2, 2);
            const material = new THREE.MeshBasicMaterial({ color: 0xff0000 });
            const marker = new THREE.Mesh(geometry, material);
            marker.position.set(x, y, z);
  
            marker.userData = station; // Store radio data in userData for access on click
  
            this.scene.add(marker);
          } else {
            console.error('Longitude or latitude is null for station:', station);
          }
        });
      },
  
      animate() {
        requestAnimationFrame(this.animate);
  
        if (this.controls) {
          this.controls.update();
        }
  
        if (this.renderer && this.scene && this.camera) {
          this.renderer.render(this.scene, this.camera);
        }
      },
      handleWindowResize() {
        this.camera.aspect = window.innerWidth / window.innerHeight;
        this.camera.updateProjectionMatrix();
        this.renderer.setSize(window.innerWidth, window.innerHeight);
      },
      togglePlayback(radio) {
        if (radio.isPlaying) {
          this.stopAudio();
        } else {
          this.playAudio(radio.url, radio);
        }
      },
      playAudio(url, radio) {
        // If another radio is already playing, stop it
        if (this.selectedStation && this.selectedStation !== radio) {
          this.stopAudio();
        }
  
        // Check if the audio URL ends with .m3u8
        if (url.endsWith('.m3u8')) {
          // Use HLS for .m3u8 streams
          if (Hls.isSupported()) {
            this.hls = new Hls();
            this.hls.loadSource(url);
            this.hls.attachMedia(this.videoPlayer); // Attach HLS to the video player
            this.hls.on(Hls.Events.MANIFEST_PARSED, () => {
              this.videoPlayer.play(); // Play the video player (hidden)
            });
          } else {
            console.error('HLS is not supported');
          }
        } else {
          // For .mp3 files, play using the HTML5 Audio element
          this.audioPlayer.src = url;
          this.audioPlayer.play();
        }
  
        // Set the selected station and update its state
        this.selectedStation = radio;
        radio.isPlaying = true;
      },
      stopAudioAndCloseModal() {
        // Stop audio playback and close the modal
        this.stopAudio();
        this.showModal = false;
      },
      stopAudio() {
        // Stop the audio player
        this.audioPlayer.pause();
        this.audioPlayer.currentTime = 0;
  
        // Stop the HLS instance and detach it from the video player
        if (this.hls) {
          this.hls.destroy();
          this.hls = null;
          this.videoPlayer.pause();
          this.videoPlayer.currentTime = 0;
        }
  
        // Update the state of the selected station
        if (this.selectedStation) {
          this.selectedStation.isPlaying = false;
        }
      },
      redirectToSite(station) {
        if (station.site) {
          window.open(station.site, '_blank'); // Open the site in a new tab
        } else {
          console.error('No site link available for the selected station.');
        }
      },
    }
  };
  </script>
  
  <style scoped>
  .site-link {
    color: blue; /* Change color as needed */
    text-decoration: underline;
    cursor: pointer;
  }
  .container {
    position: relative;
  }
  .v-dialog {
    z-index: 9999; /* Ensure modal is above Three.js scene */
  }
  </style>
  