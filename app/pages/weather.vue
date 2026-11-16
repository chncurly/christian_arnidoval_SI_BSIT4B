
<template>
  <main class="weather-page">
    <section class="weather-card">

      <header class="header">
        <div class="location">
          <h1>{{ city }}</h1>
        </div>
      </header>

      <!-- Search Bar -->
      <div class="search-container">
        <input
          v-model="searchCity"
          type="text"
          placeholder="Search city..."
          @keyup.enter="searchWeather"
        />

        <button
          @click="searchWeather"
          :disabled="loading"
        >
          {{ loading ? 'Searching...' : 'Search' }}
        </button>
      </div>

      <p v-if="loading" class="message">
        Searching weather...
      </p>

      <p v-if="errorMessage" class="error-message">
        {{ errorMessage }}
      </p>

      <section class="current-weather">
        <div class="main-info">
          <div class="condition">
            <span class="sun-icon">☼</span>
            <span>{{ weatherCondition }}</span>
          </div>

          <div class="temperature">
            {{ currentTemp }}°
          </div>

          <div class="sun-details">
            <span>↟ {{ maxTemp }}°</span>
            <span>↓ {{ minTemp }}°</span>
          </div>
        </div>
      </section>

      <!-- Hourly forecast -->
      <section class="hourly-section">
        <div class="hourly-list">
          <article
            v-for="hour in hourlyForecast"
            :key="hour.time"
            class="hour-item"
          >
            <div class="hour-temp">{{ hour.temp }}°</div>
            <div class="hour-icon">{{ hour.icon }}</div>
            <div class="hour-time">{{ hour.time }}</div>
          </article>
        </div>
      </section>

      <!-- Weekly forecast -->
      <section class="weekly-section">
        <article
          v-for="day in weeklyForecast"
          :key="day.day"
          class="day-row"
        >
          <span class="day-name">{{ day.day }}</span>
          <span class="day-icon">{{ day.icon }}</span>

          <div class="day-temps">
            <span class="day-max">{{ day.max }}°</span>
            <span class="day-min">{{ day.min }}°</span>
          </div>
        </article>
      </section>

    </section>
  </main>
</template>


<script setup>

import { ref, onMounted } from 'vue'

// Weather Variables
const city = ref('Manila')
const currentTemp = ref(0)
const maxTemp = ref(0)
const minTemp = ref(0)

const weatherCondition = ref('Sunny')

// Search Variables
const searchCity = ref('')
const loading = ref(false)
const errorMessage = ref('')

// Hourly Forecast
const hourlyForecast = [
  { time: '17:00', temp: 28, icon: '☁️' },
  { time: '18:00', temp: 26, icon: '☁️' },
  { time: '19:00', temp: 25, icon: '☁️' },
  { time: '20:00', temp: 24, icon: '☁️' },
  { time: '21:00', temp: 24, icon: '☁️' }
]

// Weekly Forecast
const weeklyForecast = [
  { day: 'tue', icon: '☀️', max: 29, min: 18 },
  { day: 'wed', icon: '☀️', max: 29, min: 18 },
  { day: 'thu', icon: '☀️', max: 31, min: 18 },
  { day: 'fri', icon: '🌤️', max: 31, min: 18 }
]

// Get Weather Function
const getWeather = async (location = 'Manila') => {

  loading.value = true
  errorMessage.value = ''

  try {

    const response = await $fetch(
      'https://api.weatherapi.com/v1/current.json',
      {
        params: {
          key: '7fd5ed1565254fdbbd305855262610',
          q: location,
          aqi: 'no'
        }
      }
    )

    if (response) {

      console.log('Successfully Fetched!', response)

      city.value = response.location.name

      currentTemp.value = Math.round(
        response.current.temp_c
      )

      weatherCondition.value =
        response.current.condition.text

      // Current API does not provide daily high/low.
      // Keep the existing forecast values for now.

      console.log(
        'Current Weather Object',
        response.current
      )

    }

  } catch (err) {

    console.error('Error Loading API', err)

    errorMessage.value =
      'City not found! Please try again.'

  } finally {

    loading.value = false

  }

}


// Search Weather Function
const searchWeather = async () => {

  if (!searchCity.value.trim()) {

    errorMessage.value =
      'Please enter a city name!'

    return

  }

  await getWeather(searchCity.value.trim())

}


// Load Default Weather
onMounted(() => {

  getWeather('Manila')

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
  justify-content: space-between;
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
  letter-spacing: 0.5px;
}

.location span {
  display: block;
  font-size: 14px;
  color: rgba(255, 255, 255, 0.72);
  margin-top: 4px;
}


/* Search Bar */

.search-container {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 10px;
  width: 100%;
  max-width: 450px;
  margin: 0 auto 20px;
}

.search-container input {
  flex: 1;
  min-width: 0;
  padding: 12px 18px;
  border: none;
  outline: none;
  border-radius: 25px;

  background: rgba(255, 255, 255, 0.2);
  color: white;

  font-size: 14px;
  backdrop-filter: blur(10px);
}

.search-container input::placeholder {
  color: rgba(255, 255, 255, 0.7);
}

.search-container button {
  padding: 12px 22px;
  border: none;
  border-radius: 25px;

  background: white;
  color: #737ac0;

  font-weight: 600;
  cursor: pointer;
  transition: 0.3s;
}

.search-container button:hover {
  background: #e0e0ff;
  transform: scale(1.05);
}

.search-container button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.message {
  text-align: center;
  font-size: 14px;
  margin-bottom: 10px;
}

.error-message {
  text-align: center;
  color: #ffe0e0;
  font-size: 14px;
  margin-bottom: 10px;
}


/* Current Weather */

.current-weather {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  padding: 20px 0;
  max-width: 800px;
  width: 100%;
  margin: 0 auto;
}

.condition {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 18px;
  color: rgba(255, 255, 255, 0.9);
}

.sun-icon {
  font-size: 24px;
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
  gap: 16px;
  font-size: 16px;
  color: rgba(255, 255, 255, 0.85);
}


.weather-info {
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 16px 20px;
  border-radius: 16px;
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(8px);
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

  border-top: 1px solid rgba(255, 255, 255, 0.2);
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);

  background: rgba(255, 255, 255, 0.08);
  border-radius: 20px;
  padding: 12px 0;
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

  border-right: 1px solid rgba(255, 255, 255, 0.15);
}

.hour-item:last-child {
  border-right: none;
}

.hour-temp {
  font-size: 16px;
  font-weight: 500;
}

.hour-icon {
  font-size: 24px;
}

.hour-time {
  font-size: 13px;
  color: rgba(255, 255, 255, 0.75);
}


/* Weekly Forecast */

.weekly-section {
  width: 100%;
  max-width: 800px;
  margin: 0 auto;

  background: rgba(255, 255, 255, 0.1);
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

.day-icon {
  font-size: 20px;
}

.day-temps {
  display: flex;
  gap: 16px;
}

.day-max {
  font-weight: 600;
}

.day-min {
  color: rgba(255, 255, 255, 0.6);
}


/* Footer */

.footer {
  padding-top: 24px;
  text-align: center;
  font-size: 11px;
  letter-spacing: 3px;
  color: rgba(255, 255, 255, 0.4);
}


/* Responsive Design */

@media (max-width: 480px) {

  .weather-card {
    padding: 24px 16px;
  }

  .temperature {
    font-size: 76px;
  }

  .search-container button {
    padding: 12px 16px;
  }

  .weekly-section {
    padding: 16px;
  }

}

</style>
