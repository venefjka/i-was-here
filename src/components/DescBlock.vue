<template>
  <div v-if="desc.length">
    {{ desc.split('.').slice(0,2).join('. ') }}.
  </div>
  <div v-else>
    {{ mapdesc.split('.').slice(0,2).join('. ')}}.
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'DescBlock',
  data() {
    return {
      desc: '',
      mapdesc:
        'Красноя́рский край — субъект Российской Федерации в Сибирском федеральном округе; относится к Восточно-Сибирскому экономическому району. Является вторым по площади субъектом России и крупнейшим из краёв. Площадь его составляет 2 366 797 км². Является третьей по величине административно-территориальной единицей в мире после Якутии и Западной Австралии. Образован 7 декабря 1934 года. Административный центр — город Красноярск. В границах практически полностью совпадает с Енисейской губернией.'
    }
  },
  methods: {
    async fetchDesc(regionName) {
      const url = 'https://kgsearch.googleapis.com/v1/entities:searchAAA'
      const apiKey = 'AIzaSyAkCAi03tTo310SnRk06NT_NL8nI0Qehe4'
      try {
        const response = await axios.get(url, {
          params: {
            query: regionName,
            languages: 'ru',
            limit: 1,
            indent: true,
            key: apiKey
          }
        })
        console.log(response)
        if (response.data && response.data.itemListElement[0]) {
          this.desc = response.data.itemListElement[0].result.detailedDescription.articleBody
        }
      } catch (error) {
        console.error('Ошибка при получении описания:', error)
      }
    }
  }
}
</script>

<style scoped>
div {
  margin: 30px;
}
</style>
