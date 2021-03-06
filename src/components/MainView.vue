<template lang="pug">
.main(v-cloak)
  .main__bg
  .main__box
    h1.main__title Ip Address Tracker
    .main__form
      input.main__input(
        type="text",
        placeholder="Search for any IP address or domain",
        v-model="ip"
      )
      a.main__button(@click="search()")
        img(src="../assets/icon-arrow.svg", alt="Icon Arrow")
    .main__detail
      .detail__item(v-for="item in detail")
        .main__subtitle
          | {{ item.title }}
        .main__text
          | {{ item.value }}
  .map-section
    l-map.map(
      v-model="zoom",
      v-model:zoom="zoom",
      :center="cords",
      :min-zoom="3",
      :max-zoom="19",
      :options="{ zoomControl: false }",
      @update:zoom="zoomUpdated",
      @update:center="centerUpdated",
      @update:bounds="boundsUpdated"
    )
      l-tile-layer(url="http://{s}.tile.osm.org/{z}/{x}/{y}.png")
      l-marker(:lat-lng="cords")
        l-tooltip
          | {{ tooltip }}
</template>

<script>
import { LMap, LTileLayer, LMarker, LTooltip } from "@vue-leaflet/vue-leaflet"

export default {
  name: 'MainView',
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LTooltip
  },
  props: {
    message: String,
  },
  data() {
    return {
      ip: null,
      detail: {
        ip: {
          title: 'IP ADDRESS',
          value: '-'
        },
        location: {
          title: 'LOCATION',
          value: '-'
        },
        timezone: {
          title: 'TIMEZONE',
          value: '-'
        },
        isp: {
          title: 'ISP',
          value: '-'
        },
      },
      zoom: 10,
      cords: [25.54384799152488, -103.52998458496477],
      bounds: null,
      tooltip: 'Lerdo, Durango, MX',
      error: false
    }
  },

  methods: {
    search() {
      if (!this.ip) {
        alert('Invalid IP')
        this.ip = null
      } else {
        fetch(`https://geo.ipify.org/api/v2/country,city?apiKey=${process.env.VUE_APP_API_KEY}&ipAddress=${this.ip}`)
          .then(response => response.json())
          .then((data) => {
            if (data.location != undefined) {
              this.detail.ip.value = data.ip
              this.detail.location.value = `${data.location.city}, ${data.location.region}, ${data.location.country}`
              this.detail.timezone.value = data.location.timezone
              this.detail.isp.value = data.isp
              this.cords = [data.location.lat, data.location.lng]
              this.tooltip = this.detail.location.value
            }
            else {
              alert('IP not found')
              this.ip = null
            }
          })
          .catch((error) => console.log(error))
      }
    },
    zoomUpdated(zoom) {
      this.zoom = zoom
    },
    centerUpdated(center) {
      this.center = center
    },
    boundsUpdated(bounds) {
      this.bounds = bounds
    }
  },
}
</script>
<style>
@import "leaflet/dist/leaflet.css";
@import url("https://fonts.googleapis.com/css2?family=Rubik:wght@400;500;700&display=swap");

@media screen and (min-width: 200px) {
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    color: #2b2b2b;
    font-family: Rubik, sans-serif;
  }

  .main {
    position: relative;
  }

  .main__bg {
    background: url(../assets/pattern-bg.png) no-repeat;
    background-size: cover;
    background-position: center;
    height: 250px;
  }

  .main__box {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    margin: 1em;
    z-index: 99;
    text-align: center;
    justify-content: center;
  }

  .main__title {
    justify-self: center;
    color: #fff;
    font-size: 1.5em;
    font-weight: 500;
  }

  .main__form {
    display: flex;
    justify-content: center;
    margin: 1.4em 0 1em;
  }

  .main__input {
    width: 100%;
    padding: 1em;
    border: none;
    border-top-left-radius: 12px;
    border-bottom-left-radius: 12px;
    outline: 0;
    color: #2b2b2b;
    font-size: 18px;
    font-weight: 400;
  }

  .main__button {
    background: #2b2b2b;
    display: flex;
    align-items: center;
    padding: 1em 1.4em;
    border: none;
    border-top-right-radius: 12px;
    border-bottom-right-radius: 12px;
    outline: 0;
    cursor: pointer;
  }

  .main__detail {
    background: #fff;
    border: none;
    border-radius: 12px;
    padding: 1em;
  }

  .main__subtitle,
  .main__text {
    display: block;
    font-weight: 700;
  }

  .main__subtitle {
    margin: 1.4em 0 0.4em;
    color: #767676;
    font-size: xx-small;
  }

  .main__subtitle:nth-child(1) {
    margin-top: 0;
  }

  .main__text {
    font-size: 18px;
  }

  .map-section {
    height: calc(100vh - 250px);
  }
  .map {
    z-index: 1;
  }
}

@media screen and (min-width: 800px) {
  .main__box {
    left: 100px;
    right: 100px;
  }

  .main__title {
    font-size: 2em;
  }

  .main__form {
    margin: 1.4em 0 2.8em;
  }

  .main__input {
    width: 50%;
  }

  .main__detail {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    text-align: left;
  }

  .detail__item {
    display: flex;
    flex-direction: column;
    padding: 1em;
    border-right: 1px solid hsla(0, 0%, 59%, 0.2);
  }

  .detail__item:nth-child(4) {
    border-right: none;
  }
}
</style>
