<template>
  <HeaderComponent />
  <div id="map" style="width: 100vw; height: 100vh"></div>
  <div class="sidebar">
    <div class="sidebar-content">
      <h2>{{ title }}</h2>
      <DescBlock ref="DescBlock" />
      <ImgBlock ref="ImgBlock" />
      <WeatherBlock ref="WeatherBlock" />
      <NewsBlock ref="NewsBlock" />
    </div>
    <button class="close-btn" @click="reset">✖</button>
  </div>
  <FooterComponent />
</template>

<script>
import * as d3 from "d3";
import * as topojson from "topojson-client";
import HeaderComponent from "./HeaderComponent.vue";
import FooterComponent from "./FooterComponent.vue";
import DescBlock from "./DescBlock.vue";
import ImgBlock from "./ImgBlock.vue";
import WeatherBlock from "./WeatherBlock.vue";
import NewsBlock from "./NewsBlock.vue";
import axios from "axios";
import { weatherRegions, regionsCity } from "../utils";

export default {
  name: "MapComponent",
  components: {
    DescBlock,
    ImgBlock,
    WeatherBlock,
    HeaderComponent,
    FooterComponent,
    NewsBlock,
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
      title: null,
      sidebarActive: false,
    };
  },
  mounted() {
    this.drawMap();
  },
  methods: {
    async getMap() {
      return await axios.get(
        "https://raw.githubusercontent.com/codeforamerica/click_that_hood/master/public/data/russia.geojson"
      );
    },

    async drawMap() {
      this.zoom = d3.zoom().scaleExtent([1, 8]).on("zoom", this.zoomed);

      this.svg = d3
        .select("#map")
        .append("svg")
        .attr("viewBox", [-50, -80, this.width, this.height])
        .attr("width", this.width)
        .attr("height", this.height)
        .attr("style", "max-width: 100%; height: 99vh;")
        .on("click", this.reset);

      this.g = this.svg.append("g");

      const response = await axios.get(
        "https://raw.githubusercontent.com/codeforamerica/click_that_hood/master/public/data/russia.geojson"
      );
      const data = response.data;

      this.projection = d3
        .geoNaturalEarth1()
        .rotate([-100, -20])
        .fitSize([this.width - 100, this.height - 100], data);
      this.path = d3.geoPath().projection(this.projection);

      const regions = this.g
        .append("g")
        .attr("fill", "rgb(155 211 232)")
        .attr("cursor", "pointer")
        .selectAll("path")
        .data(data.features)
        .join("path")
        .on("click", this.clicked)
        .on("mouseover", this.mouseover)
        .on("mouseout", this.mouseout)
        .attr("d", this.path);

      regions.append("title").text((d) => d.properties.name);
      this.g
        .append("path")
        .attr("fill", "none")
        .attr("stroke", "white")
        .attr("stroke-linejoin", "round")
        .attr("d", this.path(topojson.mesh(data, data, (a, b) => a !== b)))
        .attr("stroke-width", "10");

      this.svg.call(this.zoom);
    },

    reset() {
      console.log("jj");
      this.g.selectAll("path").transition().style("fill", null);
      this.svg
        .transition()
        .duration(750)
        .call(
          this.zoom.transform,
          d3.zoomIdentity,
          d3.zoomTransform(this.svg.node()).invert([this.width / 2, this.height / 2])
        );
      document.querySelector(".sidebar").style.display = "none";
      document.querySelector(".description").style.display = "flex";
      document.querySelector(".footer").style.display = "flex";
    },

    clicked(event, d) {
      console.log(event);
      const [[x0, y0], [x1, y1]] = this.path.bounds(d);

      event.stopPropagation();
      this.g.selectAll("path").transition().style("fill", "#F2F2EF");
      d3.select(event.currentTarget).transition().style("fill", "rgb(155 211 232)");
      this.svg
        .transition()
        .duration(1050)
        .call(
          this.zoom.transform,
          d3.zoomIdentity
            .translate(this.width / 3.5, this.height / 2)
            .scale(
              Math.min(8, 0.8 / Math.max((x1 - x0) / this.width, (y1 - y0) / this.height))
            )
            .translate(-(x0 + x1) / 2, -(y0 + y1) / 2),
          d3.pointer(event, this.svg.node())
        );

      this.viewSidebar(d);
      this.$refs.DescBlock.fetchDesc(d.properties.name);

      let weatherRegion = d.properties.name;
      const unvalidRegion = Object.keys(weatherRegions).find(
        (key) => weatherRegions[key] === weatherRegion
      );
      if (unvalidRegion) weatherRegion = regionsCity[unvalidRegion];

      this.$refs.ImgBlock.fetchImages(weatherRegion);
      this.$refs.WeatherBlock.fetchWeather(weatherRegion);
      this.$refs.NewsBlock.fetchNews(weatherRegion);
    },

    mouseover(event) {
      d3.select(event.currentTarget).style("stroke-width", "4");
    },

    mouseout(event) {
      d3.select(event.currentTarget)
        .transition()
        .style("stroke", "white")
        .style("stroke-width", "0.22");
    },

    zoomed(event) {
      const { transform } = event;
      this.g.attr("transform", transform);
      this.g.attr("stroke-width", 1 / transform.k);
    },

    viewSidebar(d) {
      document.querySelector(".sidebar").style.display = "flex";
      document.querySelector(".description").style.display = "none";
      document.querySelector(".footer").style.display = "none";
      this.title = d.properties.name;
    },
  },
};
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

.sidebar {
  position: absolute;
  top: 0;
  right: 0;
  display: none;
  width: 40vw;
  height: 100vh;
  background-color: white;
  box-shadow: -2px 0 10px rgba(0, 0, 0, 0.2);
  transition: right 0.3s ease;
  flex-direction: column;
  padding: 20px;
}

.sidebar-content {
  flex: 1;
  overflow-y: auto;
  scrollbar-width: none;
}

.close-btn {
  align-self: flex-end;
  background: none;
  border: none;
  font-size: 1.5em;
  cursor: pointer;
}

h2 {
  text-align: center;
  font-size: large;
  font-weight: bolder;
  font-family: Arial, sans-serif;
  margin: 20px 15px;
}

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
