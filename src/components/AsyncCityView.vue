<template>
    <div class="flex flex-col flex-1 items-center">
    <!-- Banner -->
      <div v-if="route.query.preview" class="text-white p-4 bg-weather-secondary w-full text-center">
            <p>
                You are currently previewing this city, click the "+" icon to start tracking this city.
            </p>
      </div>
    <!-- Weather Overview -->
      <div class="flex flex-col items-center text-white py-12">
        <h1 class="text-4xl mb-2">{{ route.params.city }}</h1>
        <p class="text-sm mb-12">
            {{
                new Date(weatherData.currentTime).toLocaleDateString(
                    "en-us",
                    {
                        weekday: "short",
                        day: "2-digit",
                        month: "long",
                    }
                )
            }}
            {{ new Date(weatherData.currentTime).toLocaleTimeString(
                "en-us",
                {
                    timeStyle: "short",
                }
            ) 
            }}
        </p>
        <p class="text-8xl mb-8">
            {{ Math.round(weatherData.main.temp) }} &deg;
        </p>
        <div class="text-center">
            <p>
                Feels like
                {{ Math.round(weatherData.main.feels_like) }}&deg;
            </p>
            <p class="capitalize">
                {{ weatherData.weather[0].description }}
            </p>
            <img class="w-[150px] h-auto" :src="`/icons/${weatherData.weather[0].icon}.png`" >
        </div>
      </div>

      <hr class="border-white border-opacity-10 border w-full"/>

      <!-- Hourly Weather -->
      <div class="max-w-screen w-full py-12">
        <div class="mx-8 text-white">
            <h2 class="mb-4">Three Hourly Weather</h2>
            <div class="flex gap-10 overflow-x-scroll">
                <div v-for="hourData in forecastData.list" :key="hourData.dt"
                class="flex flex-col gap-4 items-center">
              <p class="whitespace-nowrap text-md">
                {{ hourData.dt_txt }}
              </p>
              <img class="w-auto h-[50px] object-cover" 
              :src="`/icons/${hourData.weather[0].icon}.png`" alt="">
              <p class="text-xl">
                {{ Math.round(hourData.main.temp) }}&deg;
              </p>
            </div>
            </div>
        </div>
      </div>
i
      <hr class="border-white border-opacity-10 border w-full"/>

      <!-- Weekly Weather -->
      <div class="max-w-screen-md w-full py-12">
        <div class="mx-8 text-white">
            <h2 class="mb-4">7 Days Forecast</h2>
            <div v-for="(day, index) in dayData"
                :key="day.dt"
                class="flex items-center">
                    <p class="flex-1">
                        {{ forecastDay[index] }}
                    </p>
                    <img class="w-[50px] h-[50px] object-cover"
                    :src="`/icons/${day.weather[0].icon}.png`" alt="" />
                    <div class="flex gap-2 flex-1 justify-end">
                        <p>H: {{ Math.round(day.main.temp_max) }}</p>
                        <p>L: {{ Math.round(day.main.temp_min) }}</p>
                    </div>
                </div>
        </div>
      </div>
      <div class="flex items-center gap-2 py-12 text-white cursor-pointer duration-150 hover:text-red-500"
      @click="removeCity">
        <i class="fa-solid fa-trash"></i>
        <p>Remove City</p>
      </div>
    </div>
</template>

<script setup>
import axios from 'axios';
import { useRoute, useRouter } from 'vue-router';

const route = useRoute();
const APIKey = "a9e630b58022603ea2850628da6bdad4";
const getWeatherData = async () => {
    try {
        const weatherData = await axios.get(`https://api.openweathermap.org/data/2.5/weather?lat=${route.query.lat}&lon=${route.query.lon}&appid=${APIKey}&units=imperial`);
        const forecastData = await axios.get(`https://api.openweathermap.org/data/2.5/forecast?lat=${route.query.lat}&lon=${route.query.lon}&appid=${APIKey}&units=imperial`)
        //cal current weather
        const localOffset = new Date().getTimezoneOffset() * 60000;
        const utc = weatherData.data.dt * 1000 + localOffset;
        weatherData.data.currentTime =  utc + 1000 * weatherData.data.timezone;

        //cal hourly weather offset
        //  weatherData.data.hourly


        return [weatherData.data, forecastData.data];
    } catch (err) {
        console.log(err)
    }
};

const [weatherData, forecastData] = await getWeatherData();
const WEEK_DAYS = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday'];
const dayInAWeek = new Date().getDay();
const forecastDay = WEEK_DAYS.slice(dayInAWeek, WEEK_DAYS.length).concat(WEEK_DAYS.slice(0, dayInAWeek));

const dayData = forecastData.list.slice(0, 7);

const router = useRouter();
const removeCity = () => {
        const cities = JSON.parse(localStorage.getItem("savedCities")) ;
        const updatedCities = cities.filter((city) => city.id !== route.query.id);

        localStorage.setItem('savedCities', JSON.stringify(updatedCities));
        router.push({
            name: "home",
        })
}
console.log(weatherData);
console.log(forecastData);
</script>
