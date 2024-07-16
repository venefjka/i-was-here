<template>
  <div id="weatherInfo" v-if="weatherData">
    <img src="../assets/weather.png" id="a1"/>
    <div class="weather-icon">
      <img :src="weatherIconUrl" alt="Погода" />
    </div>
    <div class="weather-details">
      <div class="weather-temp">{{ weatherData.main.temp }}°C</div>
      <div class="weather-desc">{{ weatherData.weather[0].description }}</div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'WeatherBlock',
  data() {
    return {
      weatherData: null,
      weatherIconUrl: '',
    }
  },
  methods: {
    async fetchWeather(regionName) {
      const apiKey = '6713d4ccbb57002390bee916f32a232c'
      const url = `https://api.openweathermap.org/data/2.5/weather?q=${regionName}&appid=${apiKey}&lang=ru&units=metric`

      try {
        const response = await fetch(url)
        const data = await response.json()
        console.log(data)
        if (response.ok) {
          this.weatherData = data
          this.weatherIconUrl = `https://openweathermap.org/img/w/${data.weather[0].icon}.png`
        } else {
          console.error('Ошибка запроса погоды:', data.message)
        }
      } catch (error) {
        console.error('Ошибка выполнения запроса к API погоды', error)
      }
    }
  }
}
</script>
<style>
#a1 {
  opacity: 25%;
    height: -webkit-fill-available;
    width: 35%;
    position: relative;
    border-radius: 10px;
}
#weatherInfo {
  height: 10vh;
  display: flex;
  align-items: center;
  background-color: #f9f9f9;
  border-radius: 10px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  margin: 10px 30px;
}
.weather-icon img {
  width: 50px;
  height: auto;
}
.weather-details {
  margin-left: 10px;
}
.weather-temp {
  font-size: 1.2em;
  font-weight: bold;
}
.weather-desc {
  font-size: 0.9em;
  color: #666;
}
</style>