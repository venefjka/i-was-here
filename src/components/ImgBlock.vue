<template>
  <div v-if="images.length" class="carousel">
    <div
      v-for="(image, index) in images"
      :key="index"
      class="carousel-item"
      :style="{ backgroundImage: 'url(' + image.link + ')' }"
    >
    </div>
  </div>

  <div v-else class="carousel">
    <div class="carousel-item" style="background-image: url('https://via.placeholder.com/300')">
    </div>
    <div class="carousel-item" style="background-image: url('https://via.placeholder.com/300')">
    </div>
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
      const apiKey = 'AIzaSyAkCAi03tTo310SnRk06NT_NL8nI0Qehe4'
      const cx = 'e0fd55d846bde44ef'
      const url = `https://www.googleapis.com/customsearch/v1`

      try {
        const response = await axios.get(url, {
          params: {
            key: apiKey,
            cx: cx,
            q: title + ' фото',
            searchType: 'image',
            num: 5
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
            if (this.images.length >= 3) break
          }
        }
      } catch (error) {
        console.error('Ошибка при получении изображений:', error)
      }
    },

    checkImage(url) {
      return new Promise((resolve) => {
        var img = new Image()
        var timeout = 5000

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

<style scoped>
.carousel {
  margin: 0 30px;
  display: flex;
  scrollbar-width: none;
  overflow-x: scroll;
  margin-bottom: 30px;
  gap: 10px;
}
.carousel-item {
  flex: 0 0 auto;
  width: 300px;
  height: 200px;
  background-size: cover;
  background-position: center;
  border-radius: 10px;
  display: flex;
  align-items: flex-end;
  padding: 10px;
  color: white;
  font-weight: bold;
  text-shadow: 0 0 5px rgba(0, 0, 0, 0.7);
}
.image-title {
  background-color: rgba(0, 0, 0, 0.5);
  padding: 5px 10px;
  border-radius: 5px;
}
</style>

