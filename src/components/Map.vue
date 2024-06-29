<template>
  <div>
    <div id="map" style="width: 100vw; height: 100vh"></div>
    <div class="textbox">
      <h2>{{ title }}</h2>
      <ImgBlock ref="ImgBlock" />
      <WeatherBlock ref="WeatherBlock" />
    </div>
  </div>
</template>

<script>
import * as d3 from 'd3'
import * as topojson from 'topojson-client'
import ImgBlock from './ImgBlock.vue'
import WeatherBlock from './WeatherBlock.vue'
import axios from 'axios'
import { weatherRegions, regionsCity } from '../utils'

export default {
  name: 'MapComponent',
  components: {
    ImgBlock,
    WeatherBlock
  },
  data() {
    return {
      width: 1500,
      height: 610,
      svg: null,
      g: null,
      path: null,
      projection: null,
      zoom: null,
      title: null
    }
  },
  mounted() {
    this.drawMap()
  },
  methods: {
    async drawMap() {
      this.zoom = d3.zoom().scaleExtent([1, 8]).on('zoom', this.zoomed)

      this.svg = d3
        .select('#map')
        .append('svg')
        .attr('viewBox', [-50, -50, this.width, this.height])
        .attr('width', this.width)
        .attr('height', this.height)
        .attr('style', 'max-width: 100%; height: 99vh;')
        .on('click', this.reset)

      this.g = this.svg.append('g')

      const response = await axios.get(
        'https://raw.githubusercontent.com/codeforamerica/click_that_hood/master/public/data/russia.geojson'
      )
      const data = response.data

      this.projection = d3
        .geoNaturalEarth1()
        .rotate([-100, -20])
        .fitSize([this.width - 100, this.height - 100], data)
      this.path = d3.geoPath().projection(this.projection)

      const regions = this.g
        .append('g')
        .attr('fill', '#759242')
        .attr('cursor', 'pointer')
        .selectAll('path')
        .data(data.features)
        .join('path')
        .on('click', this.clicked)
        .attr('d', this.path)
        .attr('stroke', 'white')
        .attr('stroke-width', '0.22')

      regions.append('title').text((d) => d.properties.name)
      this.g
        .append('path')
        .attr('fill', 'none')
        .attr('stroke', 'white')
        .attr('stroke-linejoin', 'round')
        .attr('d', this.path(topojson.mesh(data, data, (a, b) => a !== b)))
        .attr('stroke-width', '10')

      this.svg.call(this.zoom)
    },

    reset() {
      this.g.selectAll('path').transition().style('fill', null)
      this.svg
        .transition()
        .duration(750)
        .call(
          this.zoom.transform,
          d3.zoomIdentity,
          d3.zoomTransform(this.svg.node()).invert([this.width / 2, this.height / 2])
        )
      document.querySelector('.textbox').style.display = 'none'
    },

    clicked(event, d) {
      const [[x0, y0], [x1, y1]] = this.path.bounds(d)

      event.stopPropagation()
      this.g.selectAll('path').transition().style('fill', '#F2F2EF')
      d3.select(event.currentTarget).transition().style('fill', '#759242')
      this.svg
        .transition()
        .duration(1050)
        .call(
          this.zoom.transform,
          d3.zoomIdentity
            .translate(this.width / 3.5, this.height / 2)
            .scale(Math.min(8, 0.8 / Math.max((x1 - x0) / this.width, (y1 - y0) / this.height)))
            .translate(-(x0 + x1) / 2, -(y0 + y1) / 2),
          d3.pointer(event, this.svg.node())
        )

        let weatherRegion = d.properties.name
        const unvalidRegion = Object.keys(weatherRegions).find(
          (key) => weatherRegions[key] === weatherRegion
        )
        if (unvalidRegion) weatherRegion = regionsCity[unvalidRegion]

        this.viewTextBox(d)
        this.$refs.ImgBlock.fetchImages(weatherRegion)
        this.$refs.WeatherBlock.fetchWeather(weatherRegion)
    },

    zoomed(event) {
      const { transform } = event
      this.g.attr('transform', transform)
      this.g.attr('stroke-width', 1 / transform.k)
    },

    viewTextBox(d) {
      document.querySelector('.textbox').style.display = 'block'
      this.title = d.properties.name
    }
  }
}
</script>

<style scoped>
body {
  font-family: Arial, sans-serif;
  width: 100%;
  height: 100vh;
  margin: 0;
}
svg {
  width: 100vw;
  height: 100vh;
}
.textbox {
  right: 5vw;
  bottom: 10vh;
  position: absolute;
  width: 37vw;
  height: 80vh;
  background-color: white;
  border-radius: 17px;
  box-shadow: 0 0 8px rgb(173, 173, 173);
  display: none;
  text-align: center;
  font-size: medium;
  font-weight: bolder;
  font-family: Arial, sans-serif;
  margin: 20px 15px;
}
h2 {
  text-align: center;
  font-size: large;
  font-weight: bolder;
  font-family: Arial, sans-serif;
  margin: 20px 15px;
}
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
