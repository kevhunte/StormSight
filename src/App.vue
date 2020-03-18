<template>
<div id="app">
  <!--<img alt="Vue logo" src="./assets/logo.png">-->
  <h5>StormSight</h5>
  <!--<HelloWorld msg="Welcome to Your Vue.js App"/>-->
  <div id="MapContainer" class="col-md-10 mx-auto">
    <div id="mapid"></div>
  </div>
</div>
</template>

<script>
//import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  components: {
    //HelloWorld
  },
  data() {
    return {
      mymap: null,
      location: null,
      icons: []
    }
  },
  mounted() {
    this.getLocation();
    this.initMap();
    //this.addMarker(40.8116, -73.9465); // icon free

  },
  methods: {
    initMap(lat, lng) {
      if (lat && lng) {
        this.mymap = L.map('mapid').setView([lat, long], 13);
      } else {
        this.mymap = L.map('mapid').setView([40.8116, -73.9465], 13); // Harlem
      }
      const attribution = '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap<a/>';
      const tileUrl = 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png';
      const tiles = L.tileLayer(tileUrl, {
        attribution
      });
      tiles.addTo(this.mymap);
    },
    addMarker(lat, lng, StormIcon) {
      if (StormIcon) {
        var marker = L.marker([lat, lng], {
          icon: StormIcon
        }).addTo(this.mymap);
      } else {
        var marker = L.marker([lat, lng]).addTo(this.mymap);
      }
      return marker;
    },
    getLocation() {
      navigator.geolocation.getCurrentPosition(pos => {
        this.location = pos.coords;
        //console.log('coords: ', pos.coords);
        this.addMarker(pos.coords.latitude, pos.coords.longitude); // adds at user's location
        //console.log(this.mymap);
        this.mymap.panTo(new L.LatLng(pos.coords.latitude, pos.coords.longitude)); // relocates on map
      }, err => {
        console.log('error - ', err);
      });
    }
  }
}
</script>

<style lang="scss" scoped>
#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
}

#mapid {
    height: 500px;
}
</style>
