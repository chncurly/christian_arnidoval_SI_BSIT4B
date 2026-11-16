
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

      <!-- Search -->
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

      <!-- Error -->
      <p v-if="error" class="error-message">
        {{ error }}
      </p>

      <!-- Loading -->
      <p v-if="loading" class="loading">
        Loading weather...
      </p>

      <!-- Current Weather -->
      <section v-if="weather" class="current-weather">
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
              ↑ {{ Math.round(weather.current.feelslike_c) }}°
            </span>
            <span>
              ↓ {{ Math.round(weather.current.wind_kph) }} km/h
            </span>
          </div>

          <div class="weather-info">
            <div>
              <span>Humidity</span>
              <span>{{ weather.current.humidity }}%</span>
            </div>

            <div>
              <span>Wind</span>
              <span>{{ weather.current.wind_kph }} km/h</span>
            </div>

            <div>
              <span>Feels Like</span>
              <span>{{ weather.current.feelslike_c }}°</span>
            </div>
          </div>

        </div>
      </section>

      <!-- Hourly Forecast -->
      <section v-if="weather" class="hourly-section">
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
      <section v-if="weather" class="weekly-section">
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

    </section>
  </main>
</template>


<script setup>
import { ref, computed, onMounted } from 'vue'

const API_KEY = 'YOUR_API_KEY'

const city = ref('Manila')
const searchCity = ref('')

const weather = ref(null)
const loading = ref(false)
const error = ref('')

const hourlyForecast = computed(() => {
  if (!weather.value) return []

  const hours = weather.value.forecast.forecastday[0].hour

  const currentHour = weather.value.current.last_updated_epoch

  return hours.filter(hour =>
    hour.time_epoch >= currentHour
  ).slice(0, 5)
})

const weeklyForecast = computed(() => {
  return weather.value?.forecast.forecastday || []
})

const getWeather = async () => {
  const searchedCity = searchCity.value.trim() || city.value

  if (!searchedCity) return

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

  } catch (err) {
    error.value = 'City not found! Please try another city.'
    console.error('Weather API Error:', err)
  } finally {
    loading.value = false
  }
}

const formatTime = (time) => {
  return time.split(' ')[1]
}

const formatDay = (date) => {
  return new Date(date + 'T00:00:00')
    .toLocaleDateString('en-US', {
      weekday: 'short'
    })
}

onMounted(() => {
  getWeather()
})
</script>


/* Search Bar */

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
  background: rgba(255, 255, 255, 0.9);
  color: #333;
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

.error-message {
  text-align: center;
  color: #ffe0e0;
  margin: 10px 0;
}

.loading {
  text-align: center;
  margin: 15px;
}

.condition img {
  width: 45px;
  height: 45px;
}

.hourly-section h3,
.weekly-section h3 {
  margin-bottom: 15px;
  font-size: 17px;
}

.hour-item img,
.day-row img {
  width: 40px;
  height: 40px;
  object-fit: contain;
}

@media (max-width: 500px) {
  .weather-card {
    padding: 24px 14px;
  }

  .temperature {
    font-size: 80px;
  }

  .search-box button {
    padding: 12px 15px;
  }

  .hourly-list {
    grid-template-columns: repeat(5, minmax(0, 1fr));
  }

  .hour-item {
    padding: 10px 2px;
  }
}
