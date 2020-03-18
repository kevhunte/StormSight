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
      icons: [],
      weatherURL: 'https://api.weather.gov/points/',
      weatherData: null
    }
  },
  mounted() {
    this.getLocation();
    this.initMap();
    //this.addMarker(40.8116, -73.9465); // icon free

  },
  methods: {
    initMap() {

      this.mymap = L.map('mapid').setView([40.8116, -73.9465], 13); // Harlem as default
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
        this.getWeatherData(pos.coords.latitude, pos.coords.longitude);
      }, err => {
        console.log('error - ', err);
      });
    },
    async getWeatherData(lat, long) {
      if (lat && long) {

        let response = await fetch(this.weatherURL + lat + ',' + long);
        let data = await response.json();
        //console.log(data);

        const forecastURL = data.properties.forecast; // pulls forecast url
        let foreRes = await fetch(forecastURL);
        let foreData = await foreRes.json();

        const weatherData = foreData.properties.periods; // data for next week
        this.weatherData = weatherData;
        console.log(weatherData);
      }
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
    height: 30rem;
}
</style>
