<template>
  <div id="weatherInfo"></div>
</template>

<script>
export default {
  name: 'WeatherBlock',
  methods: {
    async fetchWeather(regionName) {
      const apiKey = '6713d4ccbb57002390bee916f32a232c'
      const url = `https://api.openweathermap.org/data/2.5/weather?q=${regionName}&appid=${apiKey}&lang=ru&units=metric`

      try {
        const response = await fetch(url)
        const data = await response.json()
        console.log(data)
        if (response.ok) {
          const iconUrl = `https://openweathermap.org/img/w/${data.weather[0].icon}.png`
          document.getElementById('weatherInfo').innerHTML = `
          <div style="display: flex; flex-direction: column; justify-content: center">
          <img style="width: 80px; height: auto;" src="${iconUrl}" alt="Погода" />
          <div style="box-shadow: 0 0 3px rgb(236 236 236);
          font-size: 14px;
          width: 80px;
          border-radius: 15px;
          background: #ffffff;
          font-family: Arial;
          color: #535353;
          position: relative;
          top: -20px;
          height: 20px;"> ${data.main.temp}°C</div>
          </div>
          <div style="place-content: center; margin: 0px 15px;"> ${data.weather[0].description}</div>`
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
  height: 15%;
  margin: 0px 15px;
  padding: 7px 5px;
  display: flex;
  background-color: #fafafa;
  box-shadow: inset 0 0 12px rgb(240, 240, 240);
  border-radius: 14px;
  place-content: center;
  justify-content: space-evenly;
}
</style>