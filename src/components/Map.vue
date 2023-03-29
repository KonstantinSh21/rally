<template>
  <div class="map">

    <l-map style="width:100%; height: 100%" :zoom="zoom" :center="center">
      <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>
      <div v-for="marker in transcripts">
        <l-marker :lat-lng="marker.geo"></l-marker>
      </div>
    </l-map>
    <button @click="toggleMic">Start</button>
    <button @click="click">click</button>
    <div v-text="transcript"></div>

  </div>
</template>

<script>
import "leaflet";
import "leaflet/dist/leaflet.css";
const Recognition = window.SpeechRecognition || window.webkitSpeechRecognition;
const sr = new Recognition();
Recognition.lang = "ru-RU";
sr.continuous = false;

import { LMap, LTileLayer, LMarker } from "vue2-leaflet";


export default {
  name: "Map",
  components: { LMap, LTileLayer, LMarker },
  data() {
    return {
      zoom: 14,
      center: {
        lat: null,
        lng: null
      },
      url: "http://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
      attribution:'&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
      myPosition: {
        lat: null,
        lng: null
      },
      gettingLocation: false,
      transcript: '',
      transcripts: [
        {value: 'правый 3 трамплин',geo: [58.01045227050781, 56.22944259643555]},
        {value: 'правый 3 трамплин',geo: [58.151, 56.3]},
        {value: 'правый 3 трамплин',geo: [58.01045227050781, 56.2]},
        {value: 'правый 3 трамплин',geo: [58.1, 56.28]}
      ],
      isRecording: false,
    };
  },
  methods: {
    toggleMic() {
      if (this.isRecording) {
        sr.stop()
      } else {
        sr.start()
      }
    },

    CheckForCommand(result) {
    const t = result[0].transcript;
    if (t.includes('остановить')) {
      sr.stop()
      alert(new Date().toLocaleTimeString())
      setTimeout(() => sr.start(), 0)
    }
    },

    getGeoPosition() {
      navigator.geolocation.getCurrentPosition(
        pos => {
          this.center = L.latLng(pos.coords.latitude, pos.coords.longitude);
          this.gettingLocation = true;
        },
        err => {
          this.gettingLocation = false;
          this.errorStr = err.message;
        }
      );
    },

    click() {
      window.speechSynthesis.speak(utterance);
    },

    getDistanceFromLatLonInKm(lat1,lon1,lat2,lon2) {
      let R = 6371; // Radius of the earth in km
      let dLat = this.deg2rad(lat2-lat1);  // deg2rad below
      let dLon = this.deg2rad(lon2-lon1);
      let a =
        Math.sin(dLat/2) * Math.sin(dLat/2) +
        Math.cos(this.deg2rad(lat1)) * Math.cos(this.deg2rad(lat2)) *
        Math.sin(dLon/2) * Math.sin(dLon/2)
        ;
      let c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
      let d = R * c; // Distance in km
      return d;
    },

    deg2rad(deg) {
      return deg * (Math.PI/180)
    },
    startRally() {
      this.getGeoPosition();

      this.transcripts.forEach((item) => {
        console.log(item);
      })
    },
  },
  mounted() {
    this.getGeoPosition();
    sr.coutinuous = true;
    sr.interimResults = false;
    sr.maxAlternatives = true

    sr.onstart = () => {
      console.log('Start')
      this.isRecording = true;
    }

    sr.stop = () => {
        console.log('Stop')
        this.isRecording = false;
    }
    sr.onresult = (e) => {
      for (let i = 0; i < e.results.length; i++) {
			const result = e.results[i]
        if (result.isFinal) this.CheckForCommand(result)
      }
      const t = Array.from(e.results)
        .map(result => result[0])
        .map(result => result.transcript)
        .join('')
      console.log(t)
      this.transcript = t;

      const utterance = new SpeechSynthesisUtterance(t);
      window.speechSynthesis.speak(utterance);

      this.transcripts.push({value: t})

      setInterval(() => {
          sr.start();
          // this.getGeoPosition();
      }, 500);
    }

    // let distanceMOWBKK = this.getDistanceFromLatLonInKm(57.99695, 56.13571, 57.99787, 56.14722);
  },
  watch: {

  }
};
</script>

<style>

* {
  padding: 0;
  margin: 0;
}
.map {
  height: 100vh;
}
</style>



