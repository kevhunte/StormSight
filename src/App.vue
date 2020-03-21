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
      icons: [
        require('@/assets/sunny.png'),
        require('@/assets/rain.png'),
        require('@/assets/snow.png'),
        require('@/assets/cloudy.png')
      ],
      weatherURL: 'https://api.weather.gov/points/',
      weatherData: null,
      predictions: null
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

        const weatherData = foreData.properties.periods; // data for the week
        this.weatherData = weatherData;
        //console.log(weatherData);
        this.processWeatherData();
      }
    },
    async processWeatherData() {
      if (this.weatherData) { // processes weather conditions for the next 5 days
        let prediction = {}
        for (let [index, d] of this.weatherData.entries()) {
          if (index > 10) { // save time
            break;
          }
          let cond = d.shortForecast
          if (cond.includes("Sunny")) {
            prediction.hasOwnProperty('Sunny') ? prediction.Sunny.hits += 1 : prediction.Sunny = {
              "hits": 1,
              "firstSeen": index
            }
          }
          if (cond.includes("Rain")) {
            prediction.hasOwnProperty('Rain') ? prediction.Rain.hits += 1 : prediction.Rain = {
              "hits": 1,
              "firstSeen": index
            }
          }
          if (cond.includes("Snow")) {
            prediction.hasOwnProperty('Snow') ? prediction.Snow.hits += 1 : prediction.Snow = {
              "hits": 1,
              "firstSeen": index
            }
          }
          if (cond.includes("Cloudy")) {
            prediction.hasOwnProperty('Cloudy') ? prediction.Cloudy.hits += 1 : prediction.Cloudy = {
              "hits": 1,
              "firstSeen": index
            }
          }
          if (cond.includes("Clear")) {
            prediction.hasOwnProperty('Clear') ? prediction.Clear.hits += 1 : prediction.Clear = {
              "hits": 1,
              "firstSeen": index
            }
          }
        }
        console.log('processed - ', prediction);
        /*for (let p of prediction){
          makePrediction(p);
        }
        //await console.log('with weights -',prediction);
        //this.predictions = prediction;
        */
      }
    },
    makePrediction(key) { // can work on each key of predictions seperately
      // takes a key, and adds a prediction val
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
