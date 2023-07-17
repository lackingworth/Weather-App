<template>
  <div id="container" 
    :class="[typeof weather.main != 'undefined' && units === 'metric' && weather.main.temp > 16 ? 'warmC' : '',
            {warmF: typeof weather.main != 'undefined' && units === 'imperial' && weather.main.temp > 61},
            {coldF: units === 'imperial'}]">
      <main
      :class="(typeof weather.weather != 'undefined' && weather.weather[0].main === 'Clouds' ? 'wClouds' : '')">
        <div class="search-box">
          <input 
          type="text" 
          class="search-bar" 
          placeholder="Enter city name..."
          v-model="query"
          @keypress="fetchWeatherAsync">
          <button @click="fetchWeatherAsync">
            <font-awesome-icon 
              :icon="['fas', 'magnifying-glass']"  
              class="search-icon"/>
          </button>
        </div>
        
        <div class="slider">
          <label class="toggle">
            <input 
              class="toggle-input" 
              type="checkbox" 
              v-model="degreeSys"
              @click="setUnits(); unitsConverter();"/>
            <span class="toggle-label" data-off="°C" data-on="°F"></span>
            <span class="toggle-handle"></span>
          </label>
        </div>
        <Snow v-if="(typeof weather.main != 'undefined' && weather.weather[0].main === 'Snow')"/>
        <Rain v-if="(typeof weather.main != 'undefined' && weather.weather[0].main === 'Rain')"/>

        <div class="loader" v-if="loading">
        </div>
        
        <div class="error" v-if="error">
          <h1>Sorry, can't get relevant data. <br> {{ error }}</h1>
        </div>
        <div class="weather-wrap" v-if="(typeof weather.main != 'undefined' && !loading)">
          
          <div class="location-box">
            <div class="location">{{ weather.name }}, {{ weather.sys.country }}</div>
            <div class="date">{{ dateBuilder() }} </div>
          </div>
          
          <div class="weather-box">
            <div class="temp">{{ Math.round(weather.main.temp) }}{{ unitsSymb }}</div>
            <div class="weather">{{ weather.weather[0].main }}</div>
          </div>
        </div>
      
      </main>
  </div>
</template>

<script>
import Snow from './Components/Snow.vue'
import Rain from './Components/Rain.vue'

 export default {
  components: { Snow, Rain },
  data() {
    return {
      api_key: 'YOUR_API_KEY',
      api_base: 'https://api.openweathermap.org/data/2.5/',
      query: '',
      degreeSys: false,
      loading: false,
      units: 'metric',
      unitsSymb: '°C',
      temp: null,
      error: null,
      weather: {}
    }
  },

  methods: {
    setUnits() {
      if (this.degreeSys) {
        this.units = 'metric';
        this.unitsSymb = '°C';
      } else {
        this.units = 'imperial';
        this.unitsSymb = '°F'; 
      }
    },

    unitsConverter() {
      if (this.units === 'imperial' && typeof this.weather.main != 'undefined') {
        this.weather.main.temp = (Math.round(this.weather.main.temp) * (9/5)) + 32;
      } else if (this.units === 'metric' && typeof this.weather.main != 'undefined') {
        this.weather.main.temp = (Math.round(this.weather.main.temp) - 32) * (5/9);
      }
    },

    async fetchWeatherAsync(e) {
      try {
        if (e.key == "Enter" || e.pointerType === 'mouse' || e.pointerType === 'touch') {
          this.loading = true;
          const res = await fetch(`${this.api_base}weather?q=${this.query}&units=${this.units}&appid=${this.api_key}`);
          if (res.ok) {
            return await res.json()
            .then(this.setResults);
          }
          throw new Error ('Request failed');
        }
      } catch(err) {
        this.error = err;
        console.log(err);
      };
    },

    setResults(results) {
      this.loading = false;
      this.weather = results;
    },

    dateBuilder() {
      let d = new Date();
      let months = ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'];
      let days = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'];

      let day = days[d.getDay()];
      let date = d.getDate();
      let month = months[d.getMonth()];
      let year = d.getFullYear();

      return `${day} ${date} ${month} ${year}`;
    }
  }
 }


</script>

<style>
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  body {
    font-family: 'montserrat', sans-serif;
    overflow: hidden;
  }

  #container {
    background-image: url(./assets/coldbg.jpg);
    background-size: cover;
    background-position: bottom;
    transition: 0.4s;
  }

  #container.warmC {
    background-image: url(./assets/warmbg.jpg);
  }

  #container.coldF {
    background-image: url(./assets/coldbgF.jpg);
  }

  #container.warmF {
    background-image: url(./assets/warmbgF.jpg);
  }

  main {
    min-height: 100vh;
    padding: 25px;
    background-image: linear-gradient(to bottom, rgba(0, 0, 0, 0.25), rgba(0, 0, 0, 0.35)); 
  }

  .wClouds {
    background-image: linear-gradient(to bottom, rgba(0, 0, 0, 0.25), rgba(0, 0, 0, 0.75));
  }

  .search-box {
    display: flex;
    width: 100%;
    margin-bottom: 30px;
  }

  .search-box button {
    background-color: transparent;
    border: none;
  }

  .search-box .search-icon {
    min-height: 25px;
    height: 3vh;
    padding: 10px;
    margin: auto;

    cursor: pointer;
  }

  .search-box .search-bar {
    display: block;
    width: 100%;
    padding: 15px;
    
    color: #313131;
    font-size: 20px;
   
    appearance: none;
    border: none;
    outline: none;
    background: none;
    
    box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.25);
    background-color: rgba(255, 255, 255, 0.5);
    border-radius: 0px 16px 0px 16px;
    transition: 0.4s;
  }

  .search-box .search-bar:focus {
    box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
    background-color: rgba(255, 255, 255, 0.75);
    border-radius: 16px 0px 16px 0px;
  }

  .search-box .search-bar:hover {
    box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
    background-color: rgba(255, 255, 255, 0.75);
  }

  .toggle {
     position: relative;
     display: block;
     width: 100px;
     height: 40px;
     padding: 3px;
     margin: 10px;
     border-radius: 50px;
     cursor: pointer;
  }
    
  .toggle-input {
    position: absolute;
    top: 0;
    left: 0;
    opacity: 0;
  }
  
  .toggle-label {
    position: relative;
    display: block;
    height: inherit;
    font-size: 1.25rem;
    background: rgb(0, 60, 255);
    border-radius: inherit;
    box-shadow: inset 0 1px 2px rgba(0, 0, 0, 0.12), inset 0 0 3px
    rgba(0, 0, 0, 0.15);
  }

  .toggle-label:before,
  .toggle-label:after {
    position: absolute;
    top: 50%;
    color: black;
    margin-top: -.5em;
    line-height: 1;
  }
      
  .toggle-label:before {
    content: attr(data-off);
    right: 11px;
    color: #fff;
    text-shadow: 0 1px rgba(255, 255, 255, 0.5);
  }
      
  .toggle-label:after {
    content: attr(data-on);
    left: 11px;
    color: #fff;
    text-shadow: 0 1px rgba(0, 0, 0, 0.2);
    opacity: 0;
  }
      
  .toggle-input:checked~.toggle-label {
    background: green;
  }
      
  .toggle-input:checked~.toggle-label:before {
    opacity: 0;
  }
      
  .toggle-input:checked~.toggle-label:after {
    opacity: 1;
  }
      
  .toggle-handle {
    position: absolute;
    top: 4px;
    left: 4px;
    width: 38px;
    height: 38px;
    background: linear-gradient(to bottom, #FFFFFF 40%, #f0f0f0);
    border-radius: 50%;
  }
      
  .toggle-handle:before {
    position: absolute;
    top: 50%;
    left: 50%;
    margin: -6px 0 0 -6px;
    width: 16px;
    height: 16px;
  }
      
  .toggle-input:checked~.toggle-handle {
    left: 64px;
    box-shadow: -1px 1px 5px rgba(0, 0, 0, 0.2);
  }

  /* Transition*/
  .toggle-label,
  .toggle-handle {
    transition: All 0.3s ease;
    -webkit-transition: All 0.3s ease;
    -moz-transition: All 0.3s ease;
    -o-transition: All 0.3s ease;
  }

  .error {

    text-align: center;
    margin: auto;
    color:#ff3333c5;
    background-color: #aaa5a58e;
    width: 50vw;
    min-width: 350px;
    padding: 10px;
  }

  .location-box .location {
    color: #fff;
    font-size: 32px;
    font-weight: 500;
    text-align: center;
    text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
  }

  .location-box .date {
    color: #fff;
    font-size: 20px;
    font-weight: 300;
    font-style: italic;
    text-align: center;
  }

  .loader {
    margin: auto;
    border: 16px solid #f3f3f3; /* Light grey */
    border-top: 16px solid #000203; /* Blue */
    border-radius: 50%;
    width: 80px;
    height: 80px;
    animation: spin 1s linear infinite;
  }

  @keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
  }

  .weather-box { 
    text-align: center;
  }

  .weather-box .temp {
    display: inline-block;
    padding: 10px 25px;
    color: #fff;
    font-size: 102px;
    font-weight: 900;

    text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
    background-color: rgba(255, 255, 255, 0.25);
    border-radius: 16px;
    margin: 30px 0px;

    box-shadow: 3px 6px rgba(0, 0, 0, 0.25);
  }

  .weather-box .weather {
    color: #fff;
    font-size: 48px;
    font-weight: 700;
    font-style: italic;
    text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
  }

</style>
