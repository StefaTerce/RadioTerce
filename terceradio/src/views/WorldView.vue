<template>
    <div ref="container"></div>
    <v-dialog v-model="showModal" persistent max-width="600px">
        <v-card>
            <v-card-title>
                {{ selectedStation ? selectedStation.name : 'Station Details' }}
            </v-card-title>
            <v-card-text>
                <div v-if="selectedStation">
                    <p>URL: {{ selectedStation.url }}</p>
                    <p>Country: {{ selectedStation.country }}</p>
                    <!-- Add more details as needed -->
                </div>
                <div v-else>
                    <p>No station selected.</p>
                </div>
            </v-card-text>
            <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="showModal = false">Close</v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>
</template>

<script>
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import earthTexture from '../assets/8k_earth_daymap.jpg';

export default {
    name: 'ThreeJsScene',
    data() {
        return {
            camera: null,
            renderer: null,
            controls: null,
            earthRadius: 1,
            showModal: false,  // Control the visibility of the modal
            selectedStation: null, // Holds the selected radio station's data
        };
    },
    mounted() {
        this.init();
        this.animate();

        this.renderer.domElement.addEventListener('click', this.onMarkerClick, false);

        fetch('https://nl1.api.radio-browser.info/json/stations/search?limit=200&countrycode=IT&hidebroken=true&order=clickcount&reverse=true')
            .then(response => response.json())
            .then(data => {
                const italianRadioStations = data.filter(radio => radio.countrycode === 'IT');
                italianRadioStations.forEach(station => {
                    this.addMarker(station.geo_long, station.geo_lat, 0.025, station); // Pass the whole station data
                });
            })
            .catch(error => {
                console.error('Error fetching radio station data:', error);
            });

        window.addEventListener('resize', this.handleWindowResize);
    },

    beforeUnmount() {
        window.removeEventListener('resize', this.handleWindowResize);
    },
    methods: {
        init() {
            this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);

            // Manual computation to roughly position the camera towards Italy
            const phi = (90 - 42.5) * (Math.PI / 180);
            const theta = (12.5 + 180) * (Math.PI / 180);
            const distance = 1.5; // set a suitable distance that best fits your view needs
            this.camera.position.x = -distance * Math.sin(phi) * Math.cos(theta);
            this.camera.position.y = distance * Math.cos(phi);
            this.camera.position.z = distance * Math.sin(phi) * Math.sin(theta);

            this.camera.lookAt(0, 0, 0); // Ensure the camera looks at the center of the globe

            this.renderer = new THREE.WebGLRenderer();
            this.renderer.setSize(window.innerWidth, window.innerHeight);
            this.$refs.container.appendChild(this.renderer.domElement);

            this.controls = new OrbitControls(this.camera, this.renderer.domElement);
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
            mouse.y = - (event.clientY / window.innerHeight) * 2 + 1;

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

        addMarker(longitude, latitude, markerSize, radioData) {
            if (longitude !== null && latitude !== null) {
                const phi = (90 - latitude) * (Math.PI / 180);
                const theta = (longitude + 180) * (Math.PI / 180);
                const x = -this.earthRadius * Math.sin(phi) * Math.cos(theta);
                const y = this.earthRadius * Math.cos(phi);
                const z = this.earthRadius * Math.sin(phi) * Math.sin(theta);

                const geometry = new THREE.SphereGeometry(markerSize, 2, 2);
                const material = new THREE.MeshBasicMaterial({ color: 0xff0000 });
                const marker = new THREE.Mesh(geometry, material);
                marker.position.set(x, y, z);

                marker.userData = radioData; // Store radio data in userData for access on click

                this.scene.add(marker);
            } else {
                console.error('Longitude or latitude is null');
            }
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
        }
    }
};
</script>