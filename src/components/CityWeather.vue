<template>
  <main
    :class="
      typeof weather.main != 'undefined' && weather.main.temp > 17 ? 'warm' : ''
    "
  >
    <div class="tabs">
      <ul class="tabs__header">
        <li
          v-for="(tab, index) in tabs"
          :key="index"
          @click="selectTab(index)"
          :class="{
            'tab-selected': index === selectedIndex
          }"
          class="tab"
        >
          {{ tab.props.title }}
        </li>
      </ul>
      <slot></slot>
    </div>

    <div v-show="performSearch" class="search-box">
      <input
        id="searchBox"
        type="text"
        ref="input"
        class="search-bar"
        placeholder="Search..."
        v-model="query"
        @keypress="searchWeather"
        tabindex="0"
        v-focus
      />
    </div>

    <div class="refresh-box">
      <span @click="refreshWeather"><RefreshIcon /></span>
    </div>

    <ForecastResults :weather="weather" :forecast="forecast" />
  </main>
</template>

<script>
// const closest_match = require('closest-match')
import mockResponse from '../assets/mock_weather_forcast_response.json'
import {
  defineComponent,
  reactive,
  provide,
  onMounted,
  onBeforeMount,
  toRefs,
  ref,
  watchEffect
} from 'vue'
import ForecastResults from './ForecastResults.vue'
import RefreshIcon from './RefreshIcon.vue'

export default defineComponent({
  name: 'CityWeather',
  components: {
    ForecastResults,
    RefreshIcon
  },
  props: {
    performSearch: {
      type: Boolean,
      required: true
    }
  },
  setup(_, { slots, emit }) {
    const input = (ref < HTMLInputElement) | (null > null)
    watchEffect(() => {
      if (input.value) {
        input.value.focus()
      }
    })

    const state = reactive({
      selectedIndex: 0,
      tabs: [],
      count: 0,
      api_key: 'd6929a705065e90eb1385ccbcb3a15ba',
      weather_url_base: 'https://api.openweathermap.org/data/2.5/', //orig to delete
      forecast_url_base: 'https://api.openweathermap.org/data/3.0/',
      query: '',
      currentCity: '',
      weather: {},
      forecast: {},
      geolocation: {
        Beijing: { lat: 39.904, lon: 116.407 },
        Berlin: { lat: 52.52, lon: 13.405 },
        Cairo: { lat: 30.044, lon: 31.236 },
        Chicago: { lat: 41.878, lon: -87.629 },
        Delhi: { lat: 28.704, lon: -77.102 },
        Denver: { lat: 39.739, lon: -104.99 },
        Lagos: { lat: 6.524, lon: 3.379 },
        'Los Angeles': { lat: 34.055, lon: -118.243 },
        London: { lat: 51.507, lon: 0.127 },
        Madrid: { lat: 40.416, lon: -3.703 },
        'Mexico City': { lat: 19.432, lon: -99.133 },
        Moscow: { lat: 55.756, lon: 37.617 },
        'New York': { lat: 40.713, lon: -74.006 },
        'Rio de Janeiro': { lat: -22.906, lon: -43.173 },
        Rome: { lat: 41.897, lon: 12.482 },
        'San Francisco': { lat: 37.775, lon: -122.419 },
        'São Paulo': { lat: -23.556, lon: -46.64 },
        Seattle: { lat: 47.606, lon: -122.333 },
        Shanghai: { lat: 31.23, lon: 121.474 },
        'St. Louis': { lat: 38.627, lon: -90.199 },
        Tokyo: { lat: 35.676, lon: 139.65 },
        Toronto: { lat: 43.653, lon: -79.383 },
        Sydney: { lat: -33.869, lon: 151.209 }
      },
      makeLiveAPIcall: true
    })

    provide('TabsProvider', state)

    const selectTab = i => {
      state.selectedIndex = i

      const selectedTab = state.tabs[state.selectedIndex]
      fetchWeather(selectedTab.props.title)

      emit('resetSearch')
    }

    const fetchWeather = city => {
      state.currentCity = city

      const lat = state.geolocation[state.currentCity].lat
      const lon = state.geolocation[state.currentCity].lon

      if (state.makeLiveAPIcall) {
        // Wait for both current & forecast weather APIs to resolve
        Promise.all([
          fetch(
            `${state.weather_url_base}weather?q=${state.currentCity}&units=metric&appid=${state.api_key}`
          ),
          fetch(
            `${state.forecast_url_base}onecall?lat=${lat}&lon=${lon}&units=metric&appid=${state.api_key}`
          )
        ])
          // Get a JSON objec from each of the responses
          .then(responses =>
            Promise.all(responses.map(response => response.json()))
          )
          .then(setResults)
      } else {
        console.log('DEV MODE enabled — API calls not made')
        setResults(mockResponse)
      }
      state.query = ''
    }

    const searchWeather = e => {
      if (e.key == 'Enter') {
        fetchWeather(state.query)
      }
    }

    const refreshWeather = () => {
      fetchWeather(state.currentCity)
    }

    const setResults = results => {
      state.weather = results[0]
      state.forecast = results[1]
    }

    onBeforeMount(() => {
      if (slots.default) {
        state.tabs = slots
          .default()
          .filter(child => child.type.name === 'CitySearchTab')
      }
    })

    onMounted(() => {
      selectTab(0)
    })

    return { ...toRefs(state), selectTab, searchWeather, refreshWeather }
  },
  watch: {
    performSearch() {
      this.setFocus()
    }
  },
  methods: {
    setFocus() {
      setTimeout(function () {
        document.getElementById('searchBox')?.focus()
      }, 1)
    }
  }
})
</script>

<style scoped>
main {
  background-image: url('../assets/cold-bg.jpg');
  background-size: cover;
  background-position: bottom;
  transition: 0.4s;

  min-height: 100vh;
  padding: 10px;
}

main.warm {
  background-image: url('../assets/warm-bg.jpg');
}

.tabs__header {
  margin-bottom: 10px;
  list-style: none;
  padding: 0;
  display: flex;
  column-gap: 10px;
}
.tabs__header li {
  float: left;
  list-style: none;
  text-align: center;
  background-color: #fff;
  flex-basis: 100%;
  line-height: 60px;
}
.tabs__header li:hover {
  text-decoration: none;
  color: #000000;
  background-color: #33b5e5;
}
.tabs__header .tab-selected {
  background-color: #33b5e5;
}
.search-box {
  width: 100%;
  margin-bottom: 10px;
}
.search-box .search-bar {
  display: block;
  width: 100%;
  padding: 15px;

  color: #313131;
  font-size: 20px;

  appearance: none;
  border: none;
  outline: none;
  background: none;

  box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.5);
  border-radius: 0px 16px 0px 16px;
  transition: 0.4s;
}
.search-box .search-bar:focus {
  box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.75);
  border-radius: 16px 0px 16px 0px;
}
.refresh-box {
  text-align: center;
}
</style>
