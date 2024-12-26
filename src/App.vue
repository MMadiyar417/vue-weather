<template>
  <div id="app">
    <div class="container">
      <h1>Weathers App</h1>
      <input v-model="searchQuery" @input="fetchLocation" placeholder="Enter a location" />
    </div>

    <div v-if="location" class="forecast-container">
      <h2>Weather in {{ location.name }}</h2>
      <div v-if="forecast" class="current-weather">
        <div class="weather-info">
          <div>
            <p><strong>Current Weather:</strong> {{ forecast.current.description }}</p>
            <p>Temperature: {{ forecast.current.temperature }}°C</p>
            <p>Sunrise: {{ forecast.current.sunrise }}</p>
            <p>Sunset: {{ forecast.current.sunset }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import dayjs from 'dayjs';

export default {
  data() {
    return {
      searchQuery: '',
      location: null,
      forecast: null,
    };
  },
  methods: {
    async fetchLocation() {
      if (this.searchQuery.length < 3) return;

      try {
        const locationResponse = await axios.get(
          `https://geocoding-api.open-meteo.com/v1/search?name=${this.searchQuery}&count=1&language=en&format=json`
        );
        const locationData = locationResponse.data.results[0];
        if (locationData) {
          this.location = locationData;
          this.fetchWeather(locationData.latitude, locationData.longitude);
        }
      } catch (error) {
        console.error("Error fetching location data:", error);
      }
    },
    async fetchWeather(lat, lon) {
      try {
        const weatherResponse = await axios.get(
          `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&hourly=temperature_2m,weathercode&daily=sunrise,sunset&timezone=Europe%2FMoscow`
        );
        const weatherData = weatherResponse.data;

        const weatherDescriptions = {
          0: { description: "Clear", icon: "https://openweathermap.org/img/wn/01d@2x.png" },
          2: { description: "Cloudy", icon: "https://openweathermap.org/img/wn/03d@2x.png" },
          3: { description: "Overcast", icon: "https://openweathermap.org/img/wn/04d@2x.png" },
          4: { description: "Fog", icon: "https://openweathermap.org/img/wn/50d@2x.png" },
          5: { description: "Light Rain", icon: "https://openweathermap.org/img/wn/10d@2x.png" },
          6: { description: "Snow", icon: "https://openweathermap.org/img/wn/13d@2x.png" },
        }
        const weatherCode = weatherData.hourly.weathercode[0];
        const descriptionData = weatherDescriptions[weatherCode] || {
          description: "Unknown",
          icon: "https://example.com/default.png",
        };

        this.forecast = {
          current: {
            temperature: weatherData.hourly.temperature_2m[0],
            description: descriptionData.description,
            icon: descriptionData.icon,
            sunrise: dayjs(weatherData.daily.sunrise[0]).format("HH:mm"),
            sunset: dayjs(weatherData.daily.sunset[0]).format("HH:mm"),
          },
        };
      } catch (error) {
        console.error("Error fetching weather data:", error);
      }
    },
  },
};
</script>

<style >
#app {
  text-align: center;
}
body, html {
  margin: 0;
  padding: 0;
  height: 100vh;
  font-family: 'Arial', sans-serif;
  background: linear-gradient(to bottom, #87CEEB, #f0f8ff);
  display: flex;
  justify-content: center;
  align-items: center;
}

.container {
  text-align: center;
  margin-bottom: 20px;
}

input {
  width: 300px;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 16px;
}

.forecast-container {
  background: rgb(184 202 209 / 90%);
  display: flex;
  justify-content: center;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  width: 400px;
  text-align: center;
  flex-direction: column;
}

.current-weather {
  margin-top: 20px;
}

.weather-icon {
  width: 60px;
  height: 60px;
}

h1, h2 {
  color: #2c3e50;
}

p {
  margin: 5px 0;
  color: #34495e;
}
</style>
