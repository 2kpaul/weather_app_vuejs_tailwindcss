<template>
  <div id="app" :class="bgClass">
    <div class="card">
      <input type="text" placeholder="search city..." v-model="query" @keyup.enter="fetchWeather()">

      <div v-if="error" class="error">
        <div class="location">Invalid city name</div>
      </div>
      
      <div v-else>
        <div class="location-box">
          <div class="location">{{ weather.city }}, {{ weather.country }}</div>
          <div class="date">{{ formatedDate }}</div>
        </div>
      </div>    
      <div class="grid grid-cols-2 gap-2">  
        <div class="card">
          <div class="weather-box">
            <div class="temperature">{{ weather.temp }}</div>
            <div class="weather">{{ weather.weather }}</div>
            <div class="description">
              {{ weather.weather_description }} feels like {{ weather.feels_like }}
            </div>
          </div>
        </div>
        <div class="card">
          <div class="weather-box-details">  
            <div v-for="(value, name, index) in weather.details" v-bind:key="index" class="info">  
              <span>{{ formatObjectKey(name) }}:</span>
              <span>{{ value }}</span>
            </div>  
          </div>
        </div>
      </div> 
    </div>
    <div>
      <a href="https://openweathermap.org/" target="_blank" class="hover:text-sky-800">Api by openweathermap.org</a>
      <a href="https://paul2d.dev" target="_blank" class="hover:text-sky-800 block text-center">made by Paul2D</a>
    </div>
    
  </div>
  
</template>

<script>
//import HelloWorld from './components/HelloWorld.vue'
import './assets/tailwind.css';
import dayjs from 'dayjs';

export default {
  name: 'App',
  components: {
    
  }, 
  data () {
    return {
      api_key: process.env.VUE_APP_WEATHER_API_KEY,
      api_url: process.env.VUE_APP_WEATHER_API_URL,
      query: '',
      weather: {},
      date : new Date(), 
      bgClass : 'bg-gray-50',
      error : false
    }
  },
  methods: {
    fetchWeather() {
      if(this.query != '') {
        console.log(this.query);
        this.$http.get(this.api_url+'weather?q='+this.query+'&units=metric&appid='+this.api_key)
        .then(response => {
          if(response.status == 200) {
            console.log(response);
            this.weather = {
              city: response.data.name,
              country: response.data.sys.country,
              temp: Math.round(response.data.main.temp) + '℃',
              feels_like: Math.round(response.data.main.feels_like) + '℃',
              weather: response.data.weather[0].main,
              weather_description: response.data.weather[0].description,
              details : {
                temp_max: Math.round(response.data.main.temp_max) + '℃',
                temp_min: Math.round(response.data.main.temp_min) + '℃',
                wind_speed: response.data.wind.speed + 'm/s',
                visibility: (response.data.visibility / 1000) + 'Km',
                pressure: response.data.main.pressure + 'hPa',
                humidity: response.data.main.humidity + '%'
              }
            }
            this.changeBg();
            this.error = false;
          }
        })
        .catch(error => {
          console.log(error.message);
          this.error = true;
        })
      }
      this.query = '';
    },
    changeBg() {
      let temp = this.weather.temp;
      if(temp <= 20) {
        this.bgClass = 'bg-sky-200';
      } else {
        this.bgClass = 'bg-amber-200';
      }
    },
    formatObjectKey(key) {  
      const formatLabel = key.replaceAll('_', ' ').split(" ");

      return formatLabel.map((word) => { 
          return word[0].toUpperCase() + word.substring(1); 
      }).join(" ");

      
    }
  },
  mounted() {
    this.query = 'Bucharest';
    this.fetchWeather();
    this.changeBg();
    this.query = '';
  },
  computed: {
    formatedDate() {
            return dayjs(this.date).format('MMMM D YYYY, HH:mm');
        }
  },
}
</script>

