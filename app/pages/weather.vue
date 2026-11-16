
<template>
  <main class="weather-page">
    <section class="weather-card">

      <!-- Header -->
      <header class="header">
        <div class="location">
          <h1>{{ city }}</h1>
          <p v-if="weather">
            {{ weather.location.country }}
          </p>
        </div>
      </header>

      <!-- Search Bar -->
      <div class="search-box">
        <input
          v-model="searchCity"
          type="text"
          placeholder="Search city..."
          @keyup.enter="getWeather"
        />

        <button
          @click="getWeather"
          :disabled="loading"
        >
          {{ loading ? 'Searching...' : 'Search' }}
        </button>
      </div>

      <!-- Error Message -->
      <p v-if="error" class="error-message">
        {{ error }}
      </p>

      <!-- Loading -->
      <p v-if="loading" class="loading">
        Loading weather...
      </p>

      <!-- Weather Content -->
      <template v-if="weather">

        <!-- Current Weather -->
        <section class="current-weather">
          <div class="main-info">

            <div class="condition">
              <img
                :src="'https:' + weather.current.condition.icon"
                alt="Weather icon"
              />
              <span>
                {{ weather.current.condition.text }}
              </span>
            </div>

            <div class="temperature">
              {{ Math.round(weather.current.temp_c) }}°
            </div>

            <div class="sun-details">
              <span>
                Feels like
                {{ Math.round(weather.current.feelslike_c) }}°
              </span>

              <span>
                Wind {{ weather.current.wind_kph }} km/h
              </span>
            </div>

          </div>
        </section>

        <!-- Weather Details -->
        <section class="weather-info">
          <div>
            <span>Humidity</span>
            <strong>{{ weather.current.humidity }}%</strong>
          </div>

          <div>
            <span>Wind Speed</span>
            <strong>{{ weather.current.wind_kph }} km/h</strong>
          </div>

          <div>
            <span>Feels Like</span>
            <strong>
              {{ Math.round(weather.current.feelslike_c) }}°
            </strong>
          </div>

          <div>
            <span>UV Index</span>
            <strong>{{ weather.current.uv }}</strong>
          </div>
        </section>

        <!-- Hourly Forecast -->
        <section class="hourly-section">

          <h3>Hourly Forecast</h3>

          <div class="hourly-list">
            <article
              v-for="hour in hourlyForecast"
              :key="hour.time_epoch"
              class="hour-item"
            >

              <div class="hour-temp">
                {{ Math.round(hour.temp_c) }}°
              </div>

              <img
                :src="'https:' + hour.condition.icon"
                alt="Weather icon"
              />

              <div class="hour-time">
                {{ formatTime(hour.time) }}
              </div>

            </article>
          </div>

        </section>

        <!-- Weekly Forecast -->
        <section class="weekly-section">

          <h3>7-Day Forecast</h3>

          <article
            v-for="day in weeklyForecast"
            :key="day.date"
            class="day-row"
          >

            <span class="day-name">
              {{ formatDay(day.date) }}
            </span>

            <img
              :src="'https:' + day.day.condition.icon"
              alt="Weather icon"
            />

            <span class="day-condition">
              {{ day.day.condition.text }}
            </span>

            <div class="day-temps">
              <span class="day-max">
                {{ Math.round(day.day.maxtemp_c) }}°
              </span>

              <span class="day-min">
                {{ Math.round(day.day.mintemp_c) }}°
              </span>
            </div>

          </article>

        </section>

        <footer class="footer">
          WEATHER FORECAST
        </footer>

      </template>

    </section>
  </main>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

// API KEY
const API_KEY = 'YOUR_API_KEY'

// Reactive Variables
const city = ref('Manila')
const searchCity = ref('')

const weather = ref(null)

const loading = ref(false)
const error = ref('')

// Hourly Forecast
const hourlyForecast = computed(() => {
  if (!weather.value) return []

  const hours =
    weather.value.forecast.forecastday[0].hour

  const currentHour =
    weather.value.current.last_updated_epoch

  return hours
    .filter(hour => hour.time_epoch >= currentHour)
    .slice(0, 5)
})

// Weekly Forecast
const weeklyForecast = computed(() => {
  if (!weather.value) return []

  return weather.value.forecast.forecastday
})

// Get Weather Function
const getWeather = async () => {

  const searchedCity =
    searchCity.value.trim() || city.value

  if (!searchedCity) {
    error.value = 'Please enter a city name.'
    return
  }

  loading.value = true
  error.value = ''

  try {

    const response = await $fetch(
      'https://api.weatherapi.com/v1/forecast.json',
      {
        params: {
          key: API_KEY,
          q: searchedCity,
          days: 7,
          aqi: 'no',
          alerts: 'no'
        }
      }
    )

    weather.value = response

    city.value = response.location.name

    searchCity.value = ''

    console.log('Weather Successfully Loaded!', response)

  } catch (err) {

    console.error('Weather API Error:', err)

    error.value =
      err?.data?.error?.message ||
      'City not found! Please try another city.'

  } finally {

    loading.value = false

  }
}

// Format Time
const formatTime = (time) => {
  return time.split(' ')[1]
}

// Format Day
const formatDay = (date) => {
  return new Date(date + 'T00:00:00')
    .toLocaleDateString('en-US', {
      weekday: 'short'
    })
}

// Load Default Weather
onMounted(() => {
  getWeather()
})
</script>

<style scoped>

/* GLOBAL */

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

/* MAIN BACKGROUND */

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

/* WEATHER CARD */

.weather-card {
  width: 100%;
  min-height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;

  color: #fff;
  padding: 32px 24px;
}

/* HEADER */

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
  letter-spacing: 0.5px;
}

.location p {
  font-size: 14px;
  color: rgba(255,255,255,0.72);
  margin-top: 4px;
}

/* SEARCH BAR */

.search-box {
  display: flex;
  gap: 10px;
  width: 100%;
  max-width: 500px;
  margin: 10px auto 25px;
}

.search-box input {
  flex: 1;
  min-width: 0;
  padding: 14px 18px;

  border: none;
  outline: none;
  border-radius: 12px;

  font-size: 15px;
  background: rgba(255,255,255,0.95);
  color: #333;
}

.search-box input:focus {
  box-shadow: 0 0 0 3px rgba(255,255,255,0.25);
}

.search-box button {
  padding: 14px 22px;

  border: none;
  border-radius: 12px;

  background: #fff;
  color: #5967a8;

  font-weight: 600;
  cursor: pointer;
  transition: 0.3s;
}

.search-box button:hover {
  background: #dce2ff;
}

.search-box button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* ERROR AND LOADING */

.error-message {
  text-align: center;
  color: #ffe0e0;
  margin: 10px 0;
}

.loading {
  text-align: center;
  margin: 15px;
  font-size: 14px;
}

/* CURRENT WEATHER */

.current-weather {
  display: flex;
  justify-content: center;
  align-items: center;

  padding: 20px 0;

  max-width: 800px;
  width: 100%;
  margin: 0 auto;
}

.main-info {
  width: 100%;
  text-align: center;
}

.condition {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 8px;

  font-size: 18px;
  color: rgba(255,255,255,0.9);
}

.condition img {
  width: 55px;
  height: 55px;
  object-fit: contain;
}

.temperature {
  font-size: 96px;
  font-weight: 300;
  letter-spacing: -2px;
  line-height: 1;

  margin: 10px 0;
}

.sun-details {
  display: flex;
  justify-content: center;
  gap: 20px;

  font-size: 15px;
  color: rgba(255,255,255,0.85);
}

/* WEATHER DETAILS */

.weather-info {
  display: grid;
  grid-template-columns: repeat(4, 1fr);

  gap: 12px;
  width: 100%;
  max-width: 800px;

  margin: 15px auto;
  padding: 18px;

  border-radius: 16px;
  background: rgba(255,255,255,0.15);

  backdrop-filter: blur(8px);
}

.weather-info div {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;

  font-size: 13px;
}

.weather-info strong {
  font-size: 17px;
}

/* HOURLY FORECAST */

.hourly-section {
  width: 100%;
  max-width: 800px;

  margin: 20px auto;

  border-top: 1px solid rgba(255,255,255,0.2);
  border-bottom: 1px solid rgba(255,255,255,0.2);

  background: rgba(255,255,255,0.08);

  border-radius: 20px;
  padding: 18px;
}

.hourly-section h3,
.weekly-section h3 {
  margin-bottom: 15px;
  font-size: 17px;
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

.hour-item img {
  width: 40px;
  height: 40px;
  object-fit: contain;
}

.hour-time {
  font-size: 13px;
  color: rgba(255,255,255,0.75);
}

/* WEEKLY FORECAST */

.weekly-section {
  width: 100%;
  max-width: 800px;

  margin: 0 auto;

  background: rgba(255,255,255,0.1);
  border-radius: 20px;

  padding: 20px 24px;

  display: flex;
  flex-direction: column;
  gap: 14px;
}

.day-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 10px;

  font-size: 16px;
}

.day-name {
  text-transform: uppercase;
  font-weight: 500;
  width: 45px;
}

.day-row img {
  width: 40px;
  height: 40px;
  object-fit: contain;
}

.day-condition {
  flex: 1;
  font-size: 13px;
  color: rgba(255,255,255,0.8);
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

/* FOOTER */

.footer {
  padding-top: 24px;
  text-align: center;

  font-size: 11px;
  letter-spacing: 3px;

  color: rgba(255,255,255,0.4);
}

/* RESPONSIVE DESIGN */

@media (max-width: 600px) {

  .weather-card {
    padding: 24px 14px;
  }

  .temperature {
    font-size: 80px;
  }

  .weather-info {
    grid-template-columns: repeat(2, 1fr);
  }

  .search-box button {
    padding: 12px 15px;
  }

  .hourly-section {
    padding: 14px 8px;
  }

  .hour-item {
    padding: 10px 2px;
  }

  .hour-time {
    font-size: 11px;
  }

  .weekly-section {
    padding: 16px 12px;
  }

  .day-row {
    font-size: 14px;
  }

  .day-condition {
    font-size: 11px;
  }

  .day-temps {
    gap: 10px;
  }

}

</style>
