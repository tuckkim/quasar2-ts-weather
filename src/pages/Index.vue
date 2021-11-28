<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
        v-model="searchText"
        placeholder="Search city"
        dark
        borderless
        @keyup.enter="getWeatherByCity"
      >
        <template v-slot:before>
          <q-icon name="my_location" v-ripple @click="getLocation" />
        </template>
        <template v-slot:append>
          <q-btn
            round
            dense
            flat
            icon="search"
            @click.stop="getWeatherByCity"
          />
        </template>
      </q-input>
    </div>

    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div
          class="text-h4 text-weight-light"
          :title="`${weatherData.coord.lat}, ${weatherData.coord.lon}`"
        >
          {{ weatherData.name }}
        </div>
        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg">
          <span>{{ Math.round(weatherData.main.temp) }}</span>
          <span class="text-h4 relative-position degree">&deg;C</span>
        </div>
      </div>

      <div class="col text-center">
        <img
          :src="`https://openweathermap.org/img/wn/${weatherData.weather[0].icon}@4x.png`"
        />
      </div>
    </template>
    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">Quasar<br />Weather</div>
        <div class="col">
          <q-btn flat @click="getLocation">
            <q-icon left size="3em" name="my_location" />
            <div>Find my location</div>
          </q-btn>
        </div>
      </div>
    </template>

    <div class="col skyline"></div>
  </q-page>
</template>

<script lang="ts">
import { useQuasar } from 'quasar';
import { computed, defineComponent, ref } from 'vue';
import axios, { AxiosError, AxiosResponse } from 'axios';
import { WeatherDataType } from '../components/models';

export default defineComponent({
  name: 'PageIndex',
  setup() {
    const apiKey = '<api-key-here>';
    const apiUrl = 'https://api.openweathermap.org/data/2.5/weather';
    const $q = useQuasar();
    const searchText = ref<string | null>(null);
    const weatherData = ref<WeatherDataType | null>(null);
    const _coord = { latitude: 0, longitude: 0 };

    const bgClass = computed(() => {
      if (weatherData.value) {
        if (weatherData.value.weather[0].icon.endsWith('n')) {
          return 'bg-night';
        }
        return 'bg-day';
      }
      return null;
    });

    function _toggleLoad(isLoad: boolean) {
      isLoad ? $q.loading.show() : $q.loading.hide();
    }
    function getLocation() {
      _toggleLoad(true);
      if ($q.platform.is.electron) {
        void axios
          .get('https://freegeoip.app/json/')
          .then((res: AxiosResponse<typeof _coord>) => {
            _coord.latitude = res.data.latitude;
            _coord.longitude = res.data.longitude;
            void getWeatherByCoords();
            _toggleLoad(false);
          })
          .catch((err: AxiosError) => {
            const errObj = err.toJSON() as { message: string };
            $q.notify({
              color: 'red-5',
              textColor: 'white',
              icon: 'warning',
              message: `${errObj.message}`,
            });
            _toggleLoad(false);
          });
      } else {
        navigator.geolocation.getCurrentPosition(
          (position) => {
            console.log(position);
            _coord.latitude = position.coords.latitude;
            _coord.longitude = position.coords.longitude;
            void getWeatherByCoords();
            _toggleLoad(false);
          },
          (err) => {
            $q.notify({
              color: 'red-5',
              textColor: 'white',
              icon: 'warning',
              message: `(${err.code}) ${err.message}`,
            });
            _toggleLoad(false);
          }
        );
      }
    }

    async function getWeatherByCoords() {
      try {
        const res = await axios({
          url: `${apiUrl}?lat=${_coord.latitude}&lon=${_coord.longitude}&appid=${apiKey}&units=metric`,
        });
        weatherData.value = res.data as WeatherDataType;
        console.log(weatherData.value);
      } catch (err) {
        console.error(err);
      }
    }
    async function getWeatherByCity() {
      if (searchText.value) {
        try {
          _toggleLoad(true);
          const res = await axios({
            url: `${apiUrl}?q=${searchText.value}&appid=${apiKey}&units=metric`,
          });
          weatherData.value = res.data as WeatherDataType;
          console.log(weatherData.value);
        } catch (err) {
          console.error(err);
        }
        _toggleLoad(false);
      } else {
        $q.notify({
          color: 'red-5',
          textColor: 'white',
          icon: 'warning',
          message: 'Please enter city to search',
        });
      }
    }

    return {
      bgClass,
      searchText,
      weatherData,
      getLocation,
      getWeatherByCity,
    };
  },
});
</script>

<style lang="scss" scoped>
.degree {
  top: -44px;
}
.q-page {
  background: #093028; /* fallback for old browsers */
  background: -webkit-linear-gradient(
    to bottom,
    #237a57,
    #093028
  ); /* Chrome 10-25, Safari 5.1-6 */
  background: linear-gradient(
    to bottom,
    #237a57,
    #093028
  ); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
}
.q-page.bg-day {
  background: #f79d00; /* fallback for old browsers */
  background: -webkit-linear-gradient(
    to top,
    #64f38c,
    #f79d00
  ); /* Chrome 10-25, Safari 5.1-6 */
  background: linear-gradient(
    to top,
    #64f38c,
    #f79d00
  ); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
}
.q-page.bg-night {
  background: #000000; /* fallback for old browsers */
  background: -webkit-linear-gradient(
    to bottom,
    #856439,
    #241402
  ); /* Chrome 10-25, Safari 5.1-6 */
  background: linear-gradient(
    to bottom,
    #856439,
    #241402
  ); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
}

.skyline {
  flex: 0 0 100px;
  background: url(../assets/skyline2.png);
  background-size: contain;
  background-position: center bottom;
  margin: 0;
  padding: 0;
}
</style>
