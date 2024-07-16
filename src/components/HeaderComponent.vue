<template>
  <header class="description">
      <p class="logo">Discover Russia ▬▬▬</p>
      <p>
        Этот сайт предназначен для интерактивного исследования регионов России. Нажмите на любой
        регион, чтобы увидеть дополнительную информацию, изображения и текущую погоду.
      </p>
  </header>
</template>

<style scoped>
.description {
  display: flex;
  position: absolute;
  top: 5vh;
  left: 10vw;
  padding: 10px;
  max-width: 550px;
  font-family: Arial, sans-serif;
  font-size: 14px;
  line-height: 1.5;
  text-align: justify;
}
.logo {
  place-content: center;
  font-variant-caps: unicase;
  font-weight: 900;
  text-align: right;
  margin-right: 15px;
}
</style>
