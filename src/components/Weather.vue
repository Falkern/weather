<template>
  <div class="weather-app">
    <h1>What's My Weather</h1>
    <input
      v-model="city"
      @keyup.enter="debounceFetchWeather"
      placeholder="Enter city"
    />
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
    const debounceTimeout = ref<ReturnType<typeof setTimeout> | null>(null); // For debounce

    const fetchWeather = async () => {
      if (city.value.trim() === '') {
        errorMessage.value = 'Please enter a city name.';
        return;
      }

      const apiKey = import.meta.env.VITE_OPENWEATHER_API_KEY; // Use environment variable
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
      debounceTimeout.value = setTimeout(fetchWeather, 300); // Debounce for 300ms
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
  width: 100px; /* Set the desired width */
  height: auto; /* Maintain aspect ratio */
}
</style>
