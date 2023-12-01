<template>
  <main class="container text-white py-8">
    <!-- Search Input -->
    <div class="pt-8 mb-10 relative">
      <input
        type="text"
        v-model="searchQuery"
        @input="getSearchResults"
        placeholder="Search for a city or state..."
        class="text-2xl py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
      />
      <ul
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
        v-if="mapboxSearchResults"
      >
        <p v-if="searchError" class="px-2 py-2">
          <i class="fa-solid fa-triangle-exclamation" style="color: #ffd22e"></i> Sorry, something
          went wrong, please try again.
        </p>
        <p v-if="!searchError && mapboxSearchResults.length === 0" class="px-2 py-2">
          <i class="fa-solid fa-circle-question" style="color: #ffd22e"></i> No results match your
          query, try a different place.
        </p>

        <template v-else>
          <li
            v-for="searchResult in mapboxSearchResults"
            :key="searchResult.id"
            class="px-2 py-2 cursor-pointer"
            @click="previewCity(searchResult)"
          >
            {{ searchResult.place_name }}
          </li>
        </template>
      </ul>
    </div>
    <!-- City List Saved -->
    <div class="flex flex-col gap-4">
      <Suspense>
        <CityList />
        <template #fallback>
          <CityCardSkeleton />
        </template>
      </Suspense>
    </div>
  </main>
</template>

<script setup>
import { Suspense, ref } from 'vue'
import axios from 'axios'
import { useRouter } from 'vue-router'
import CityList from '../components/CityList.vue'
import CityCardSkeleton from '../components/CityCardSkeleton.vue'

const mapboxAPIKey = import.meta.env.VITE_MAPBOX_API_KEY
const searchQuery = ref('')
const queryTimeout = ref(null)
const mapboxSearchResults = ref(null)
const searchError = ref(null)
const router = useRouter()

// fetch data from api from search input
const getSearchResults = () => {
  clearTimeout(queryTimeout.value)
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== '') {
      try {
        const result = await axios.get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKey}&types=place`
        )
        mapboxSearchResults.value = result.data.features
        return
      } catch (err) {
        searchError.value = true
      }
    }
    mapboxSearchResults.value = null
  }, 300)
}

const previewCity = (searchResult) => {
  const [city, state] = searchResult.place_name.split(',')
  router.push({
    name: 'cityView',
    params: {
      state: state ? state.replace(/\s/g, '') : city.replace(/\s/g, ''),
      city: city
    },
    query: {
      lat: searchResult.geometry.coordinates[1],
      lng: searchResult.geometry.coordinates[0],
      preview: true
    }
  })
}
</script>
