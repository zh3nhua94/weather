<template>
  <div class="flex flex-col flex-1 items-center">
    <!-- Banner -->
    <div v-if="route.query.preview" class="text-white p-4 bg-weather-secondary w-full text-center">
      <p>You are currently previewing this city, click the "+" icon to start tracking this city</p>
    </div>

    <!-- Weather Overview -->
    <div class="flex flex-col items-center text-white py-12">
      <h1 class="text-4xl mb-2">{{ route.params.city }}</h1>
      <p class="text-sm mb-12">
        {{
          new Date(weatherData.currentTime).toLocaleDateString('en-us', {
            weekday: 'short',
            day: '2-digit',
            month: 'long'
          })
        }}
        {{
          new Date(weatherData.currentTime).toLocaleTimeString('en-us', {
            timeStyle: 'short'
          })
        }}
      </p>
      <p class="text-6xl mb-8">{{ Math.round(weatherData.current.temp - 273.15) }}&deg;C</p>
      <div class="-ml-2 flex flex-row-reverse gap-2 items-center">
        <div>
          <p>
            Feels like
            {{ Math.round(weatherData.current.feels_like - 273.15) }} &deg;C
          </p>
          <p class="capitalize">
            {{ weatherData.current.weather[0].description }}
          </p>
        </div>

        <img
          class="w-[80px] h-auto"
          :src="`http://openweathermap.org/img/wn/${weatherData.current.weather[0].icon}@4x.png`"
          alt=""
        />
      </div>
    </div>

    <hr class="border-white border-opacity-10 border w-full" />

    <!-- Hourly Weather -->
    <div class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">Hourly Weather</h2>
        <div class="flex gap-10 overflow-x-scroll">
          <div
            v-for="hourData in weatherData.hourly"
            :key="hourData.dt"
            class="flex flex-col gap-4 items-center"
          >
            <p class="whitespace-nowrap text-md">
              {{
                new Date(hourData.currentTime).toLocaleTimeString('en-us', {
                  hour: 'numeric'
                })
              }}
            </p>
            <img
              class="w-auto h-[50px] object-cover"
              :src="`http://openweathermap.org/img/wn/${hourData.weather[0].icon}@2x.png`"
              alt=""
            />
            <p class="text-xl pb-6">{{ Math.round(hourData.temp - 273.15) }}&deg;C</p>
          </div>
        </div>
      </div>
    </div>

    <hr class="border-white border-opacity-10 border w-full" />

    <!-- Weekly Weather -->
    <div class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">7 Day Forecast</h2>
        <div v-for="day in weatherData.daily" :key="day.dt" class="flex items-center">
          <p class="flex-1">
            {{
              new Date(day.dt * 1000).toLocaleDateString('en-us', {
                weekday: 'long'
              })
            }}
          </p>
          <img
            class="w-[50px] h-[50px] object-cover"
            :src="`http://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png`"
            alt=""
          />
          <div class="flex gap-2 flex-2 sm:flex-1 justify-end">
            <p class="mr-2">High: {{ Math.round(day.temp.max - 273.15) }}&deg;C</p>
            <p>Low: {{ Math.round(day.temp.min - 273.15) }}&deg;C</p>
          </div>
        </div>
      </div>
    </div>

    <div
      class="flex items-center gap-2 py-12 text-white cursor-pointer duration-150 hover:text-red-500"
      @click="removeCity"
      v-if="route.query.id"
    >
      <i class="fa-solid fa-trash"></i>
      <p>Remove City</p>
    </div>
  </div>
</template>

<!-- Script -->
<script setup>
import axios from 'axios'
import { useRoute, useRouter } from 'vue-router'

const route = useRoute()
const router = useRouter()
const getWeatherData = async () => {
  try {
    const weatherData = await axios.get(
      `https://api.openweathermap.org/data/3.0/onecall?lat=${route.query.lat}&lon=${
        route.query.lng
      }&exclude={part}&appid=${import.meta.env.VITE_OPENWEATHERMAP_API_KEY}`
    )

    // cal current date & time
    const localOffset = new Date().getTimezoneOffset() * 60000
    const utc = weatherData.data.current.dt * 1000 + localOffset
    weatherData.data.currentTime = utc + 1000 * weatherData.data.timezone_offset

    // cal hourly weather offset
    weatherData.data.hourly.forEach((hour) => {
      const utc = hour.dt * 1000 + localOffset
      hour.currentTime = utc + 1000 * weatherData.data.timezone_offset
    })

    //Delay Load
    await new Promise((res) => setTimeout(res, 500))

    return weatherData.data
  } catch (err) {
    console.log(err)
  }
}

const weatherData = await getWeatherData()

const removeCity = () => {
  const cities = JSON.parse(localStorage.getItem('savedCities'))
  const updatedCities = cities.filter((city) => city.id !== route.query.id)
  localStorage.setItem('savedCities', JSON.stringify(updatedCities))
  router.push({
    name: 'home'
  })
}
</script>

<style scoped>
::-webkit-scrollbar {
  height: 4px;
  width: 4px;
  background: gray;
}
::-webkit-scrollbar-thumb:horizontal {
  background: #000;
  border-radius: 10px;
}
</style>
