<template>
  <div class="today-wrapper">
    <GoogleMap @maploaded="getWeather"></GoogleMap>
    <div v-if="dataReady">
      <div class="today-top">
        <div class="left">
          <div>{{ weatherData.cityName }}</div>
          <div>{{ todayDate }}</div>
          <div>
            <img :src="weatherData.icon" />
          </div>
          <div>
            {{ weatherData.weatherMain }}
            <small>({{ weatherData.weather }})</small>
          </div>
        </div>
        <div class="right">
          <div class="current">
            <div class="temp">{{ weatherData.temp }}</div>
            <div>
              <span class="temp">{{ weatherData.minTemp }}</span> -
              <span class="temp">{{ weatherData.maxTemp }}</span>
            </div>
          </div>
          <div @click="toggleInfo">
            {{ infoVisible ? "Hide Info" : "More Info" }}
          </div>
        </div>
      </div>

      <div class="info" v-if="infoVisible">
        <div class="single">
          <header>{{ weatherData.humidity }}<small>%</small></header>
          <article>Humidity</article>
        </div>
        <div class="single">
          <header>{{ weatherData.wind }}<small>m/s</small></header>
          <article>Wind</article>
        </div>

        <div class="single">
          <header>{{ weatherData.sunrise }}</header>
          <article>Sunrise</article>
        </div>
        <div class="single">
          <header>{{ weatherData.sunset }}</header>
          <article>Sunset</article>
        </div>
        <div class="single">
          <header>{{ weatherData.pressure }}</header>
          <article>Pressure</article>
        </div>
      </div>

      <div class="today-bottom">
        <div class="next-seven">
          <header>Next 7 Days Forecast</header>
          <div class="forecast">
            <div class="forecast-item" v-for="(r, n) in nextSevenData" :key="n">
              <header>{{ r.date }}</header>
              <img :src="r.icon" />
              <div class="temp">{{ r.temp }}</div>
              <div>{{ r.weather }}</div>
            </div>
          </div>
        </div>
        <div class="next-seven">
          <header>Last 5 Days Forecast</header>
          <div class="forecast">
            <div class="forecast-item" v-for="(r, n) in lastFiveData" :key="n">
              <header>{{ r.date }}</header>
              <img :src="r.icon" />
              <div class="temp">{{ r.temp }}</div>
              <div>{{ r.weather }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div v-else class="loader-wrapper">
      <div class="theloader"></div>
    </div>
  </div>
</template>
<script>
import { onMounted, ref } from "vue";
import axios from "axios";

import GoogleMap from "./GoogleMap.vue";
export default {
  setup() {
    name: "TodayComp";
    const dataReady = ref(false);
    const infoVisible = ref(false);
    const weatherData = ref({});
    const nextSevenData = ref([]);
    const lastFiveData = ref([]);
    const todayDate = ref(new Date().toDateString());
    const toggleInfo = () => {
      infoVisible.value = !infoVisible.value;
    };

    const getWeather = async (e) => {
      dataReady.value = false;
      try {
        const response = await axios.get(
          `https://api.openweathermap.org/data/2.5/weather?lat=${e.lat}&lon=${e.lng}&appid=a14e48e67dc50e832f32c9d0b7c7c413&units=metric`
        );

        const sunrise = new Date(response.data.sys.sunrise * 1000);
        const sunset = new Date(response.data.sys.sunset * 1000);

        weatherData.value = {
          cityName: response.data.name,
          weatherMain: response.data.weather[0].main,
          weather: response.data.weather[0].description,
          temp: response.data.main.temp,
          minTemp: response.data.main.temp_min,
          maxTemp: response.data.main.temp_max,
          pressure: response.data.main.pressure,
          humidity: response.data.main.humidity,
          wind: response.data.wind.speed,
          sunrise: sunrise.getHours() + ":" + sunrise.getMinutes(),
          sunset: sunset.getHours() + ":" + sunset.getMinutes(),
          icon: `https://openweathermap.org/img/wn/${response.data.weather[0].icon}@2x.png`,
        };

        // get next seven days data
        const responseNextSeven = await axios.get(
          `https://api.openweathermap.org/data/2.5/onecall?lat=${e.lat}&lon=${e.lng}&appid=a14e48e67dc50e832f32c9d0b7c7c413&units=metric`
        );

        nextSevenData.value = responseNextSeven.data.daily.map((item) => {
          const d = new Date(item.dt * 1000);
          const month = d.toLocaleString("default", { month: "long" });

          return {
            date: month + " " + d.getDate(),
            temp: item.temp.morn,
            weather: item.weather[0].main,
            icon: `https://openweathermap.org/img/wn/${item.weather[0].icon}@2x.png`,
          };
        });
      } catch (e) {
        console.log("api error", e);
      }

      // get last 5 days data
      let dateList = [];
      for (let i = 1; i <= 5; i++) {
        let d = new Date();
        d.setDate(d.getDate() - i);
        const dt = d / 1000;

        dateList.push(parseInt(dt));
      }

      // making API Calls concurrent
      const requests = dateList.map((dt) =>
        axios.get(
          `https://api.openweathermap.org/data/2.5/onecall/timemachine?dt=${dt}&lat=${e.lat}&lon=${e.lng}&appid=a14e48e67dc50e832f32c9d0b7c7c413&units=metric&only_current=true`
        )
      );

      try {
        const result = await Promise.all(requests);

        lastFiveData.value = result.map((item) => {
          const d = new Date(item.data.current.dt * 1000);
          const month = d.toLocaleString("default", { month: "long" });

          return {
            date: month + " " + d.getDate(),
            temp: item.data.current.temp,
            weather: item.data.current.weather[0].main,
            icon: `https://openweathermap.org/img/wn/${item.data.current.weather[0].icon}@2x.png`,
          };
        });
      } catch (e) {
        console.log("api error", e);
      }
      dataReady.value = true;
    };

    onMounted(() => {});
    return {
      infoVisible,
      toggleInfo,
      getWeather,
      weatherData,
      todayDate,
      nextSevenData,
      lastFiveData,
      dataReady,
    };
  },
  components: { GoogleMap },
};
</script>

<style lang="scss" scoped>
@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.theloader {
  border: 4px solid #f3f3f3;
  border-top: 4px solid #ec6e4c;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  animation: spin 2s linear infinite;
}

.loader-wrapper {
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 50px;
  margin-bottom: 50px;
}

.temp {
  &::after {
    content: "O";
    font-size: 6px;
    position: absolute;
  }
}
.today-wrapper {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: auto;
  color: #fff;
  justify-content: space-between;

  .today-top {
    display: flex;
    width: 100%;
    justify-content: space-between;
    flex-wrap: wrap;
    .left {
      display: flex;

      flex-direction: column;
      letter-spacing: 2px;
      margin-top: 10px;

      > div:nth-child(1) {
        font-size: 40px;
        margin-bottom: 5px;
      }

      > div:nth-child(2) {
        font-size: 20px;
        margin-bottom: 0px;
      }

      > div:nth-child(3) {
        margin-bottom: 0px;
        img {
          width: 80px;
        }
      }

      > div:nth-child(4) {
        font-size: 20px;
        font-weight: bold;
        text-transform: capitalize;
      }
    }

    .right {
      display: flex;

      flex-direction: column;
      justify-content: space-between;
      align-content: flex-end;

      .current {
        margin-top: 10px;
        > div:nth-child(1) {
          display: flex;
          font-size: 70px;

          &::after {
            position: relative;
            font-size: 20px;
          }
        }
      }

      > div:nth-child(2) {
        cursor: pointer;
        text-decoration: underline;
        margin-top: 10px;
      }
    }
  }

  .info {
    display: flex;
    width: 100%;
    margin-top: 50px;
    gap: 30px;
    flex-wrap: wrap;

    .single {
      header {
        font-size: 30px;
        line-height: 1.3;
      }

      article {
        font-size: 15px;
      }
    }
  }

  .today-bottom {
    display: flex;
    flex-direction: column;
    width: 100%;
    justify-content: space-between;

    .next-seven {
      display: flex;
      width: 100%;
      flex-direction: column;
      margin-top: 50px;

      .forecast {
        display: flex;
        width: 100%;
        margin-top: 20px;
        gap: 20px;
        flex-wrap: wrap;

        .forecast-item {
          display: flex;
          flex-direction: column;
          justify-content: center;
          align-items: center;
          width: 100px;
          padding: 10px;
          background-color: rgba(200, 198, 197, 0.2);
          gap: 10px;
          border-radius: 6px;

          img {
            width: 80%;
          }
        }
      }
    }
  }
}
</style>