<template>
  <div class="weather-wrap" v-if="typeof weather.main != 'undefined'">
    <div
      class="location-box"
      :class="
        typeof weather.main != 'undefined' && weather.main.temp > 17
          ? 'light-text'
          : 'dark-text'
      "
    >
      <div class="location">{{ weather.name }}, {{ weather.sys.country }}</div>
      <div class="date">{{ getFormattedCurrentDate() }}</div>
    </div>

    <div
      class="weather-box"
      :class="
        typeof weather.main != 'undefined' && weather.main.temp > 17
          ? 'light-text'
          : 'dark-text'
      "
    ></div>

    <div class="hourly-box">
      <h2>Next Hours</h2>
      <ul>
        <li v-for="hour in forecast.hourly">
          <span>{{ Math.round(hour.temp) }}°</span>
          <div class="percent-rain">
            {{
              typeof hour.rain != 'undefined' && hour.rain['1h'] != 0
                ? `${Math.round(hour.rain['1h'] * 100)}%`
                : '0%'
            }}
          </div>
          <img :src="getWeatherIcon(hour.weather[0].icon)" />
          <span>{{ getFormattedTime(hour.dt) }}</span>
        </li>
      </ul>
    </div>
    <div class="daily-box">
      <h2>Next 5 Days</h2>
      <ul>
        <li v-for="o in 5" :key="o">
          <div>
            <img :src="getWeatherIcon(forecast.daily[o].weather[0].icon)" />
          </div>
          <div class="daily-summary">
            <div>
              <strong>{{ formattedTimestamp(forecast.daily[o].dt) }}</strong>
            </div>
            <div>{{ forecast.daily[o].summary }}</div>
          </div>
          <div class="high-low">
            <span>{{ Math.round(forecast.daily[o].temp.max) }}°</span>
            <span>{{ Math.round(forecast.daily[o].temp.min) }}°</span>
          </div>
        </li>
      </ul>
    </div>

    <div class="last-updated">
      Last updated on {{ formattedTimestamp(weather.dt) }}
      {{ getFormattedTime(weather.dt) }}
    </div>
  </div>
</template>

<script>
export default {
  name: 'ForecastResults',
  props: {
    weather: { type: {}, required: false },
    forecast: { type: {}, required: false }
  },
  methods: {
    formatRainHour(rain1h) {
      console.log(rain1h)
      return rain1h != 0 ? 100 * rain1h : 0
    },
    getWeatherIcon(icon) {
      // append @2x to double the size
      return `https://openweathermap.org/img/wn/${icon}.png`
    },
    getFormattedTime(unix_timestamp) {
      // Create a new JavaScript Date object based on the timestamp
      // multiplied by 1000 so that the argument is in milliseconds, not seconds
      const date = new Date(unix_timestamp * 1000)

      // Hours part from the timestamp
      let hours = date.getHours()

      // Minutes part from the timestamp
      const minutes = '0' + date.getMinutes()

      const ampm = hours >= 12 ? 'PM' : 'AM'

      hours = hours % 12
      hours = hours ? hours : 12 // the hour '0' should be '12'

      // Will display time in 10:30:23 format
      var formattedTime = hours + ':' + minutes.substr(-2) + ' ' + ampm

      return formattedTime
    },
    formattedTimestamp(unix_timestamp) {
      const date = new Date(unix_timestamp * 1000)

      // returns formatted date without year
      return date.toDateString().replace(/([A-Z])|\b[0-9]{4}\b/g, ' $1')
    },
    getFormattedCurrentDate() {
      let d = new Date()
      let months = [
        'January',
        'February',
        'March',
        'April',
        'May',
        'June',
        'July',
        'August',
        'September',
        'October',
        'November',
        'December'
      ]
      let days = [
        'Sunday',
        'Monday',
        'Tuesday',
        'Wednesday',
        'Thursday',
        'Friday',
        'Saturday'
      ]

      let day = days[d.getDay()]
      let date = d.getDate()
      let month = months[d.getMonth()]

      return `${day}, ${month} ${date}`
    }
  }
}
</script>

<style scoped>
.location-box .location {
  font-size: 32px;
  font-weight: 500;
  text-align: center;
  text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
}

.weather-box {
  text-align: center;
  padding-bottom: 1rem;
}

.weather-box .temp {
  display: inline-block;
  padding: 10px 25px;
  font-size: 102px;
  font-weight: 900;

  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.25);
  border-radius: 16px;
  margin: 15px 0px;

  box-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}

.weather-box .weather {
  font-size: 48px;
  font-weight: 700;
  font-style: italic;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}

.dark-text > div {
  color: #333333;
}
.light-text > div {
  color: #fff;
}

.location-box .date {
  font-size: 20px;
  font-weight: 300;
  font-style: italic;
  text-align: center;
  margin-top: 0.5rem;
}

.hourly-box ul {
  margin-bottom: 10px;
  list-style: none;
  padding: 0;
  display: flex;
  column-gap: 10px;
  overflow-x: scroll;
}
.hourly-box li {
  float: left;
  list-style: none;
  text-align: center;
  background-color: #fff;
  flex-basis: 100%;
}
.hourly-box {
  margin-bottom: 15px;
  list-style: none;
  padding: 0;
}

.daily-box li {
  list-style: none;
  background-color: #fff;
  display: flex;
}

.daily-summary {
  width: 70%;
  text-align: center;
  display: flex;
  flex-direction: column;
}

.daily-summary div {
  align-items: center;
  flex-basis: 100%;
}

.high-low {
  width: 20%;
  display: flex;
  align-items: center;
}
.high-low span {
  display: flex;
  flex-basis: 100%;
  justify-content: space-between;
}

h2 {
  color: white;
  margin: 8px 0;
}

.last-updated {
  color: #fff;
  text-align: right;
  margin-top: 5px;
}
.percent-rain {
  color: #add8e6;
}
</style>
