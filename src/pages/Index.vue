<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
        v-model="search"
        placeholder="Seach"
        dark
        borderless
        @keyup.enter="getWeatherSearch()"
      >
        <template v-slot:before>
          <q-icon name="my_location" @click="getLocation()" />
        </template>

        <template v-slot:append>
          <q-btn round dense flat icon="search" @click="getWeatherSearch()" />
        </template>
      </q-input>
    </div>
    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">
          {{ this.weatherData.name }}
        </div>
        <div class="text-h6 text-weight-light">
          {{ this.weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ Math.round(this.weatherData.main.temp) }}</span>
          <span class="text-h4 relative-position degree">&deg;C</span>
        </div>
      </div>
      <div class="col text-center">
        <img
          :src="
            `https://openweathermap.org/img/wn/${this.weatherData.weather[0].icon}@2x.png`
          "
          alt=""
        />
      </div>
    </template>
    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-white text-center text-weight-thin">
          Quasar Weather App
        </div>
        <q-btn class="col" flat @click="getLocation()">
          <q-icon left size="3em" name="my_location"></q-icon>
          <div>Find my location</div>
        </q-btn>
      </div>
    </template>
    <div class="col skyline"></div>
  </q-page>
</template>

<script>
export default {
  name: "PageIndex",
  data() {
    return {
      search: "",
      weatherData: null,
      lat: null,
      lon: null,
      apiUrl: "https://api.openweathermap.org/data/2.5/weather",
      apiKey: "398892e9f01c97d34cbe97ac5ee09aba",
      unit: "metric"
    };
  },
  computed: {
    bgClass() {
      if (this.weatherData)
        if (this.weatherData.weather[0].icon.endsWith("d")) return "bg-Day";
        else return "bg-Night";
    }
  },
  methods: {
    getLocation() {
      this.$q.loading.show();

      if (this.$q.platform.is.electron) {
        this.$axios.get(`https://freegeoip.app/json/`).then(response => {
          this.lat = response.data.latitude;
          this.lon = response.data.longitude;
          this.getWeather();
        });
      } else {
        navigator.geolocation.getCurrentPosition(position => {
          console.log("position", position);
          this.lat = position.coords.latitude;
          this.lon = position.coords.longitude;
          this.getWeather();
        });
      }
    },
    getWeather() {
      this.$q.loading.show();
      this.$axios(
        `${this.apiUrl}?lat=${this.lat}&lon=${this.lon}&appid=${this.apiKey}&units=${this.unit}`
      ).then(response => {
        console.log(response);
        this.weatherData = response.data;
        this.$q.loading.hide();
      });
    },
    getWeatherSearch() {
      this.$q.loading.show();
      this.$axios(
        `${this.apiUrl}?q=${this.search}&appid=${this.apiKey}&units=${this.unit}`
      ).then(response => {
        console.log(response);
        this.weatherData = response.data;
        this.$q.loading.hide();
      });
    }
  }
};
</script>

<style lang="sass">
.q-page
    background: linear-gradient(to bottom, #136a8a, #267871)
    &.bg-Night
      background: linear-gradient(to bottom, #232526, #414345)
    &.bg-Day
      background: linear-gradient(to bottom, #00b4db, #0083b0)

.degree
  top: -44px

.skyline
  flex: 0 0 100px
  background: url("../assets/skyline.png")
  background-size: contain
  background-position: center bottom
</style>
