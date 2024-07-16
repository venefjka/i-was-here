<template>
    <div class="news-block">
      <h3>Последние новости</h3>
      <ul>
        <li v-for="article in articles" :key="article.url">
          <a :href="article.url" target="_blank">{{ article.title }}</a>
        </li>
      </ul>
    </div>
</template>
  
<script>
  import axios from 'axios'
  
  export default {
    name: 'NewsBlock',
    data() {
      return {
        articles: []
      }
    },
    methods: {
      async fetchNews(region) {
        try {
          const response = await axios.get(`https://newsapi.org/v2/everything?q=${region}&pageSize=2&apiKey=c299e08683d042b2a2e6739d31c0b489`)
          this.articles = response.data.articles
        } catch (error) {
          console.error('Error fetching news:', error)
        }
      }
    }
  }
  </script>
  
  <style scoped>
  .news-block {
    margin: 30px;
  }
  .news-block h3 {
    margin-bottom: 10px;
  }
  .news-block ul {
    list-style: none;
    padding: 0;
  }
  .news-block li {
    margin-bottom: 8px;
  }
  .news-block a {
    color: #007bff;
    text-decoration: none;
  }
  .news-block a:hover {
    text-decoration: underline;
  }
  </style>
  