<template>
<div id="app">
  <!--<img alt="Vue logo" src="./assets/logo.png">-->
  <h5>StormSight</h5>
  <!--<HelloWorld msg="Welcome to Your Vue.js App"/>-->
  <div id="tempContainer" class="col-md-10 mx-auto">
    <h6 v-if="this.temp">
      {{this.temp}}&deg; F <br>
      {{this.shortDesc}}
    </h6>
    <h6 v-else>
      This application needs GPS services to function correctly.
    </h6>
  </div>
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
      predictions: null,
      temp: null,
      shortDesc: null
    }
  },
  mounted() {
    this.getLocation();
    this.initMap();
    //this.addMarker(40.8116, -73.9465); // icon free

  },
  watch: {
    'predictions': function(val) {
      if (val) {
        //console.log(val); // weather data
        let lat = this.location.latitude;
        let long = this.location.longitude;
        let offset = 0.01;
        if (val.hasOwnProperty('Sunny')) {
          if (val.Sunny.hasOwnProperty('guess')) {
            this.addMarker(lat + offset, long, [this.icons[0], 'Sunny', val.Sunny.guess]);
          }
        }
        if (val.hasOwnProperty('Rain')) {
          if (val.Rain.hasOwnProperty('guess')) {
            this.addMarker(lat, long + offset, [this.icons[1], 'Rain', val.Rain.guess]);
          }
        }
        if (val.hasOwnProperty('Snow')) {
          if (val.Snow.hasOwnProperty('guess')) {
            this.addMarker(lat - offset, long, [this.icons[2], 'Snow', val.Snow.guess]);
          }
        }
        if (val.hasOwnProperty('Cloudy')) {
          if (val.Cloudy.hasOwnProperty('guess')) {
            this.addMarker(lat, long - offset, [this.icons[3], 'Cloudy', val.Cloudy.guess]);
          }
        }
      }
    }
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
        // figure out best way to do this
        /*
        let icon = L.icon({
          iconUrl: StormIcon[0],
          iconSize: [38, 38] // figure out how to control opacity
        });*/

        let icon = L.divIcon({
          html: '<img style="opacity:' + StormIcon[2] + '; width:2.2rem; height:2.2rem;" src="' + StormIcon[0] + '">',
          className: null,
          iconSize: [38, 38]
        })
        var marker = L.marker([lat, lng], {
          icon: icon
        }).addTo(this.mymap).bindPopup(StormIcon[1]);
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
        this.temp = weatherData[0].temperature;
        this.shortDesc = weatherData[0].shortForecast;
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
        //console.log('processed - ', prediction);
        for (let i in prediction) {
          this.makePrediction(prediction[i]);
        }
        //await console.log('with weights -', prediction);
        this.predictions = prediction; // trigger watcher to make storm icons for map
      }
    },
    makePrediction(cond) {
      if (cond.firstSeen < 4) { // only make prediction if condition going to occur within two days.
        cond.guess = +((2 * cond.hits) / (10 + cond.firstSeen)).toPrecision(2);
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
