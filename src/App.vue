<template>
  <div id="app">
    <header class="header">
      <div class="nav-container">
        <span>Steve's Simple Weather App</span>
        <span @click="performSearchClick()"><SearchIcon /></span>
      </div>
    </header>

    <CityWeather
      :performRefresh="performRefresh"
      :performSearch="performSearch"
      @resetSearch="handleResetSearch"
    >
      <CitySearchTab title="Rio de Janeiro"></CitySearchTab>
      <CitySearchTab title="Beijing"></CitySearchTab>
      <CitySearchTab title="Los Angeles"></CitySearchTab>
    </CityWeather>
  </div>
</template>

<script>
import { defineComponent } from 'vue'
import CitySearchTab from './components/CitySearchTab.vue'
import CityWeather from './components/CityWeather.vue'
import SearchIcon from './components/SearchIcon.vue'

export default defineComponent({
  name: 'app',
  components: {
    CitySearchTab,
    CityWeather,
    SearchIcon
  },
  data() {
    return {
      performSearch: false
    }
  },
  methods: {
    performRefreshClick() {
      this.performRefreshCalled = !this.performRefreshCalled
    },
    performSearchClick() {
      this.performSearch = !this.performSearch
    },
    handleResetSearch() {
      this.performSearch = false
    }
  }
})
</script>

<style>
#app {
  background-image: linear-gradient(
    to bottom,
    rgba(0, 0, 0, 0.25),
    rgba(0, 0, 0, 0.75)
  );
}

header {
  background: #4d4ddb;
  color: white;

  padding: 0.5rem 0;
}

.nav-container {
  display: flex;
  align-items: center;
  justify-content: space-between;

  width: 90%;
  margin: 0 auto;
}
</style>
