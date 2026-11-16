
<template>
  <main class="weather-page">
    <section class="weather-card">

      <!-- Header -->
      <header class="header">
        <div class="location">
          <h1>{{ city }}</h1>
          <span>Weather Forecast</span>
        </div>
      </header>

      <!-- Search -->
      <div class="search-container">
        <input
          v-model="searchCity"
          type="text"
          placeholder="Enter city..."
          @keyup.enter="getWeather"
        />

        <button
          @click="getWeather"
          :disabled="loading || !searchCity.trim()"
        >
          {{ loading ? 'Loading...' : 'Search' }}
        </button>
      </div>

      <p v-if="errorMessage" class="error-message">
        {{ errorMessage }}
      </p>

      <!-- Current Weather -->
      <section v-if="weather" class="current-weather">
        <div class="main-info">

          <div class="condition">
            <img
              :src="'https:' + weather.condition.icon"
              alt="Weather icon"
            />
            <span>{{ weather.condition.text }}</span>
          </div>

          <div class="temperature">
            {{ Math.round(weather.temp_c) }}°
          </div>

          <div class="sun-details">
            <span>
              Feels like: {{ Math.round(weather.feelslike_c) }}°
            </span>
          </div>

          <div class="weather-info">
            <div>
              <span>Humidity</span>
              <span>{{ weather.humidity }}%</span>
            </div>

            <div>
              <span>Wind Speed</span>
              <span>{{ weather.wind_kph }} km/h</span>
            </div>

            <div>
              <span>UV Index</span>
              <span>{{ weather.uv }}</span>
            </div>
          </div>

        </div>
      </section>

      <!-- Hourly Forecast -->
      <section
        v-if="hourlyForecast.length"
        class="hourly-section"
      >
        <h3>Hourly Forecast</h3>

        <div class="hourly-list">
          <article
            v-for="hour in hourlyForecast"
            :key="hour.time"
            class="hour-item"
          >
            <div class="hour-temp">
              {{ Math.round(hour.temp_c) }}°
            </div>

            <div class="hour-icon">
              <img
                :src="'https:' + hour.condition.icon"
                alt="Weather"
              />
            </div>

            <div class="hour-time">
              {{ hour.time.split(' ')[1] }}
            </div>
          </article>
        </div>
      </section>

      <!-- Weekly Forecast -->
      <section
        v-if="weeklyForecast.length"
        class="weekly-section"
      >
        <h3>Weekly Forecast</h3>

        <article
          v-for="day in weeklyForecast"
          :key="day.date"
          class="day-row"
        >
          <span class="day-name">
            {{ day.day }}
          </span>

          <span class="day-icon">
            <img
              :src="'https:' + day.condition.icon"
              alt="Weather"
            />
          </span>

          <div class="day-temps">
            <span class="day-max">
              {{ Math.round(day.max) }}°
            </span>

            <span class="day-min">
              {{ Math.round(day.min) }}°
            </span>
          </div>
        </article>
      </section>

      <p v-if="loading" class="loading">
        Loading weather...
      </p>

    </section>
  </main>
</template>

<script setup>
import { ref, onMounted } from 'vue'

// API KEY
const API_KEY = 'YOUR_API_KEY'

// Variables
const city = ref('Manila')
const searchCity = ref('Manila')

const weather = ref(null)

const hourlyForecast = ref([])
const weeklyForecast = ref([])

const loading = ref(false)
const errorMessage = ref('')

// Get Weather Function
const getWeather = async () => {

  if (!searchCity.value.trim()) {
    errorMessage.value = 'Please enter a city name!'
    return
  }

  loading.value = true
  errorMessage.value = ''

  try {

    const response = await $fetch(
      'https://api.weatherapi.com/v1/forecast.json',
      {
        params: {
          key: API_KEY,
          q: searchCity.value.trim(),
          days: 7,
          aqi: 'no',
          alerts: 'no'
        }
      }
    )

    // Update City
    city.value = response.location.name

    // Update Current Weather
    weather.value = response.current

    // Hourly Forecast
    const currentHour = response.location.localtime
      .split(' ')[1]

    const currentDate = response.location.localtime
      .split(' ')[0]

    const todayHours =
      response.forecast.forecastday[0].hour

    hourlyForecast.value = todayHours
      .filter(hour => {
        return (
          hour.time.split(' ')[1] >= currentHour
        )
      })
      .slice(0, 5)

    // Weekly Forecast
    weeklyForecast.value =
      response.forecast.forecastday.map(day => {

        const date = new Date(day.date + 'T00:00:00')

        return {
          date: day.date,
          day: date.toLocaleDateString('en-US', {
            weekday: 'short'
          }),
          max: day.day.maxtemp_c,
          min: day.day.mintemp_c,
          condition: day.day.condition
        }

      })

  } catch (error) {

    weather.value = null
    hourlyForecast.value = []
    weeklyForecast.value = []

    errorMessage.value =
      'City not found! Please try another city.'

    console.error('Weather API Error:', error)

  } finally {

    loading.value = false

  }
}

// Load Default Weather
onMounted(() => {
  getWeather()
})
</script>

<style scoped>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

.weather-page {
  width: 100%;
  min-height: calc(100vh - 60px);
  display: flex;
  font-family: 'Inter', sans-serif;

  background: linear-gradient(
    155deg,
    rgba(103, 126, 177, 0.94),
    rgba(111, 121, 192, 0.97) 55%,
    #737ac0
  );
}

.weather-card {
  width: 100%;
  min-height: 100%;
  display: flex;
  flex-direction: column;
  color: #fff;
  padding: 32px 24px;
}

/* Header */
.header {
  display: flex;
  justify-content: center;
  align-items: center;
  padding-bottom: 20px;
}

.location {
  text-align: center;
}

.location h1 {
  font-size: 28px;
  font-weight: 600;
}

.location span {
  display: block;
  font-size: 14px;
  color: rgba(255,255,255,0.72);
  margin-top: 4px;
}

/* Search */
.search-container {
  display: flex;
  justify-content: center;
  gap: 8px;
  width: 100%;
  max-width: 500px;
  margin: 10px auto 20px;
}

.search-container input {
  flex: 1;
  min-width: 0;
  padding: 12px 15px;
  border: none;
  outline: none;
  border-radius: 8px;
  font-size: 14px;
}

.search-container button {
  padding: 12px 18px;
  border: none;
  border-radius: 8px;
  background: white;
  color: #737ac0;
  font-weight: 600;
  cursor: pointer;
}

.search-container button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.error-message {
  text-align: center;
  color: #ffe0e0;
  margin-bottom: 15px;
}

/* Current Weather */
.current-weather {
  display: flex;
  justify-content: center;
  align-items: flex-end;
  padding: 20px 0;
  max-width: 800px;
  width: 100%;
  margin: 0 auto;
}

.main-info {
  width: 100%;
}

.condition {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 18px;
}

.condition img {
  width: 35px;
  height: 35px;
}

.temperature {
  font-size: 96px;
  font-weight: 300;
  letter-spacing: -2px;
  line-height: 1;
  margin: 10px 0;
}

.sun-details {
  font-size: 16px;
  color: rgba(255,255,255,0.85);
}

/* Weather Info */
.weather-info {
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 16px 20px;
  margin-top: 20px;
  border-radius: 16px;
  background: rgba(255,255,255,0.15);
  font-size: 14px;
}

.weather-info div {
  display: flex;
  justify-content: space-between;
  gap: 16px;
}

/* Hourly Forecast */
.hourly-section {
  width: 100%;
  max-width: 800px;
  margin: 20px auto;
  border-top: 1px solid rgba(255,255,255,0.2);
  border-bottom: 1px solid rgba(255,255,255,0.2);
  background: rgba(255,255,255,0.08);
  border-radius: 20px;
  padding: 16px;
}

.hourly-section h3,
.weekly-section h3 {
  font-size: 16px;
  margin-bottom: 15px;
}

.hourly-list {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
}

.hour-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  padding: 12px 0;
  border-right: 1px solid rgba(255,255,255,0.15);
}

.hour-item:last-child {
  border-right: none;
}

.hour-temp {
  font-size: 16px;
  font-weight: 500;
}

.hour-icon img {
  width: 35px;
  height: 35px;
}

.hour-time {
  font-size: 13px;
  color: rgba(255,255,255,0.75);
}

/* Weekly Forecast */
.weekly-section {
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
  background: rgba(255,255,255,0.1);
  border-radius: 20px;
  padding: 16px 24px;
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.day-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 16px;
}

.day-name {
  text-transform: uppercase;
  font-weight: 500;
  width: 60px;
}

.day-icon img {
  width: 30px;
  height: 30px;
}

.day-temps {
  display: flex;
  gap: 16px;
}

.day-max {
  font-weight: 600;
}

.day-min {
  color: rgba(255,255,255,0.6);
}

.loading {
  text-align: center;
  margin: 15px;
}

/* Responsive */
@media (max-width: 480px) {
  .weather-card {
    padding: 24px 16px;
  }

  .temperature {
    font-size: 80px;
  }

  .weekly-section {
    padding: 16px;
  }
}
</style>
