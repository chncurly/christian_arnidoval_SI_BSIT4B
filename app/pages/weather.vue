<template>
  <main class="weather-page">
    <section class="weather-card">
      
      <header class="header">
        <div class="location">
          <h1>{{ city }}</h1>
          
        </div>
      </header>

      <section class="current-weather">
        <div class="main-info">
          <div class="condition">
            <span class="sun-icon">☼</span>
            <span>Sunny</span>
          </div>

          <div class="temperature">28°</div>

          <div class="sun-details">
            <span>↟ 29°</span>
            <span>↓ 18°</span>
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
            <div class="hour-temp">{{ hour.temp }}</div>
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
import { ref } from 'vue'

const city = ref('Manila')

const hourlyForecast = [
  { time: '17:00', temp: 28, icon: '☁️' },
  { time: '18:00', temp: 26, icon: '☁️' },
  { time: '19:00', temp: 25, icon: '☁️' },
  { time: '20:00', temp: 24, icon: '☁️' },
  { time: '21:00', temp: 24, icon: '☁️' }
]

const weeklyForecast = [
  { day: 'tue', icon: '☀️', max: 29, min: 18 },
  { day: 'wed', icon: '☀️', max: 29, min: 18 },
  { day: 'thu', icon: '☀️', max: 31, min: 18 },
  { day: 'fri', icon: '🌤️', max: 31, min: 18 }
]
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
</style>