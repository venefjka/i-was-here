<template>
  <div class="image-container">
    <img class="image"
      src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQVC50hZCTVNiRJj8LwK5CbdV9qAOYiF1HCng&s"
    />
    <img class="image"
      src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQVC50hZCTVNiRJj8LwK5CbdV9qAOYiF1HCng&s"
    />
  </div>
  <div v-if="images.length" class="image-container">
    <img
      v-for="(image, index) in images"
      :key="index"
      :src="image.link"
      :alt="image.title"
      class="image"
    />
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'ImgBlock',
  data() {
    return {
      images: []
    }
  },
  methods: {
    async fetchImages(title) {
      console.log(title)
      this.images = []
      //   const apiKey = 'AIzaSyAAJfKP6J2nYt_q9bo3xFSIU_AClIxa3Q4'
      //   const cx = '82def8350e3ad4b2e'
      const apiKey = 'AIzaSyAkCAi03tTo310SnRk06NT_NL8nI0Qehe4'
      const cx = 'e0fd55d846bde44ef'
      const url = `https://www.googleapis.com/customsearch/v1AAA`

      try {
        const response = await axios.get(url, {
          params: {
            key: apiKey,
            cx: cx,
            q: title + ' фото',
            searchType: 'image',
            num: 3
          }
        })
        console.log(response)
        const items = response.data.items

        for (let item of items) {
          const isImageOk = await this.checkImage(item.link)
          if (isImageOk) {
            this.images.push({
              link: item.link,
              title: item.title
            })
            if (this.images.length >= 2) break 
          }
        }
      } catch (error) {
        console.error('Ошибка при получении изображений:', error)
      }
    },

    checkImage(url) {
      return new Promise((resolve) => {
        var img = new Image()
        var timeout = 5 

        var timer = setTimeout(() => {
          img.src = 'about:blank' // Отмена загрузки
          resolve(false)
        }, timeout)

        img.onload = function () {
          clearTimeout(timer)
          resolve(true)
        }

        img.onerror = function () {
          clearTimeout(timer)
          resolve(false)
        }

        img.src = url
      })
    }
  }
}
</script>

<style>
.image-container {
  display: flex;
  flex-wrap: wrap;
  /* height: 120px; */
  /* max-height: 120px; */
  margin-bottom: 30px;

  .image {
    width: 50%;
    height: auto;
  }
}
</style>
