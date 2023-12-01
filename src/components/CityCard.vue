<template>
  <div class="flex py-6 px-3 bg-weather-secondary rounded-md shadow-md cursor-pointer">
    <div class="flex flex-col flex-1">
      <h2 class="text-xl sm:text-3xl">
        {{ city.city }}
      </h2>
      <h3>{{ city.state }}</h3>
    </div>
    <div class="flex flex-col gap-2">
      <div class="flex flex-row items-center self-end justify-between w-full">
        <img
          class="w-[50px] h-auto object-cover"
          :src="`http://openweathermap.org/img/wn/${city.weather.weather[0].icon}@4x.png`"
          alt=""
        />
        <p class="text-xl sm:text-3xl">{{ Math.round(city.weather.main.temp) }}&deg;C</p>
      </div>
      <div class="flex flex-row gap-2 text-right">
        <span class="text-xs mr-1">
          H<span class="hidden sm:inline">igh</span>:
          {{ Math.round(city.weather.main.temp_max) }}&deg;C
        </span>
        <span class="text-xs">
          L<span class="hidden sm:inline">ow</span>:
          {{ Math.round(city.weather.main.temp_min) }}&deg;C
        </span>
      </div>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  city: {
    type: Object,
    default: () => {}
  }
})

console.log(props.city)
</script>
