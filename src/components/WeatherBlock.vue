<template>
  <div id="weatherInfo" v-if="weatherData">
    <div style="display: flex; flex-direction: column; align-items: center;">
      <img style="width: 80px; height: auto;" :src="weatherIconUrl" alt="Погода" />
      <div class="weather-temp">{{ weatherData.main.temp }}°C</div>
    </div>
    <div class="weather-desc">{{ weatherData.weather[0].description }}</div>
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
#weatherInfo {
  justify-content: center;
  align-items: center;
}

.weather-temp {
  box-shadow: 0 0 3px rgb(236, 236, 236);
  font-size: 14px;
  width: 80px;
  border-radius: 15px;
  background: #ffffff;
  font-family: Arial;
  color: #535353;
  position: relative;
  top: -20px;
  height: 20px;
  display: flex;
  justify-content: center;
}

.weather-desc {
  place-content: center;
  margin: 0px 15px;
  text-align: center;
}
</style>