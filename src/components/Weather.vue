<template>
  <div class="weather-app">
    <h1>What's My Weather</h1>
    <button @click="debounceFetchWeather">Search</button>
    <div v-if="loading" class="loading">Loading...</div>
    <div v-if="errorMessage" class="error">{{ errorMessage }}</div>
    <div v-if="weather" class="weather-info">
      <h2>{{ weather.name }}</h2>
      <p>{{ weather.weather[0].description }}</p>
      <img :src="iconUrl" alt="Weather icon" />
      <p>{{ weather.main.temp }} °C</p>
      <p>Humidity: {{ weather.main.humidity }}%</p>
      <p>Wind Speed: {{ weather.wind.speed }} m/s</p>
    </div>
    <footer class="footer">
      <a href="https://github.com/FALKERN" target="_blank" class="glowing-text">
        Made by FALKERN
      </a>
    </footer>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed } from 'vue';
import axios from 'axios';

interface WeatherData {
  name: string;
  weather: { description: string; icon: string }[];
  main: { temp: number; humidity: number };
  wind: { speed: number };
}

export default defineComponent({
  name: 'Weather',
  setup() {
    const city = ref('');
    const weather = ref<WeatherData | null>(null);
    const errorMessage = ref('');
    const loading = ref(false);
    const debounceTimeout = ref<ReturnType<typeof setTimeout> | null>(null);

    const fetchWeather = async () => {
      if (city.value.trim() === '') {
        errorMessage.value = 'Please enter a city name.';
        return;
      }

      const apiKey = import.meta.env.VITE_OPENWEATHER_API_KEY;
      loading.value = true;
      errorMessage.value = '';
      try {
        const response = await axios.get(`https://api.openweathermap.org/data/2.5/weather`, {
          params: {
            q: city.value,
            units: 'metric',
            appid: apiKey,
          },
        });
        weather.value = response.data;
      } catch (error) {
        if (axios.isAxiosError(error)) {
          errorMessage.value = error.response?.data?.message || 'An unexpected error occurred.';
        } else {
          errorMessage.value = 'Network error. Please try again.';
        }
      } finally {
        loading.value = false;
      }
    };

    const debounceFetchWeather = () => {
      if (debounceTimeout.value) {
        clearTimeout(debounceTimeout.value);
      }
      debounceTimeout.value = setTimeout(fetchWeather, 300);
    };

    const iconUrl = computed(() => {
      if (weather.value && weather.value.weather.length > 0) {
        const iconCode = weather.value.weather[0].icon;
        return `http://openweathermap.org/img/wn/${iconCode}@2x.png`;
      }
      return '';
    });

    return {
      city,
      weather,
      errorMessage,
      loading,
      debounceFetchWeather,
      iconUrl,
    };
  },
});
</script>

<style scoped>
.weather-info {
  margin-top: 20px;
}

.error {
  color: red;
}

.loading {
  color: blue;
}

.weather-info img {
  width: 100px;
  height: auto;
}

.footer {
  text-align: center;
  margin-top: 20px;
}

.glowing-text {
  color: #fff;
  text-decoration: none;
  font-size: 18px;
  font-weight: bold;
  animation: glow 1.5s ease-in-out infinite alternate;
}

@keyframes glow {
  from {
    text-shadow: 0 0 1px #ffffff, 0 0 2px #ffffff, 0 0 3px #ffffff, 0 0 4px #ffffff, 0 0 5px #ffffff;
  }

  to {
    text-shadow: 0 0 2px #d2d2d2, 0 0 4px #d2d2d2, 0 0 6px #d2d2d2, 0 0 8px #d2d2d2, 0 0 10px #d2d2d2;
  }
}
</style>