<template>
  <div class="search-wrapper">
    <input type="text" v-model="searchInput" placeholder="Enter Location" />
    <button @click.prevent="searchMap">Search</button>
  </div>
  <div class="map-wrapper">
    <div @click="mapVisible = !mapVisible">
      {{ mapVisible ? "Hide Map" : "Show Map" }}
    </div>
    <div v-show="!ready">Map is Loading...</div>
    <div v-show="ready && mapVisible">
      <div ref="mapDiv" class="the-map" style="width: 100%; height: 50vh"></div>
    </div>
  </div>
</template>
<script>
import { ref, onMounted, onUnmounted } from "vue";
import { Loader } from "@googlemaps/js-api-loader";

const GOOGLE_MAPS_API_KEY = "AIzaSyC7muR9EKO38ud9ZC557R-y-94i_qwQ3Kc";

export default {
  emits: ["maploaded"],
  setup(props, { emit }) {
    const isSupported = "navigator" in window && "geolocation" in navigator;
    const currentPosition = ref({
      lat: 0,
      lng: 0,
    });

    const loader = new Loader({ apiKey: GOOGLE_MAPS_API_KEY });
    const mapDiv = ref(null);
    const searchInput = ref(null);
    const ready = ref(false);
    const mapVisible = ref(true);

    const map = ref(null);
    let marker = null;
    const geocoder = ref(null);
    let clickListener = null;
    let watcher = null;

    const clear = () => {
      marker.setMap(null);
    };

    const geocode = async (request) => {
      clear();

      try {
        const response = await geocoder.value.geocode(request);
        const { results } = response;
        map.value.setCenter(results[0].geometry.location);
        map.value.setZoom(17);
        marker.setPosition(results[0].geometry.location);
        marker = new google.maps.Marker({
          map: map.value,
          position: results[0].geometry.location,
        });
        searchInput.value = results[0].formatted_address;
        emit("maploaded", {
          lat: results[0].geometry.location.lat(),
          lng: results[0].geometry.location.lng(),
        });
      } catch (e) {
        console.log("Geocode was not successful: " + e);
      }
    };

    const searchMap = async () => {
      await geocode({ address: searchInput.value });
    };

    onMounted(async () => {
      ready.value = false;
      if (isSupported) {
        watcher = navigator.geolocation.watchPosition((position) => {
          currentPosition.value = {
            lat: position.coords.latitude,
            lng: position.coords.longitude,
          };
        });
      }

      // load map
      await loader.load();
      map.value = new google.maps.Map(mapDiv.value, {
        center: currentPosition.value,
        zoom: 10,
      });

      geocoder.value = new google.maps.Geocoder();

      marker = new google.maps.Marker({
        map: map.value,
      });

      await geocode({ location: currentPosition.value });

      clickListener = map.value.addListener("click", async (e) => {
        await geocode({ location: e.latLng });
        currentPosition.value = { lat: e.latLng.lat(), lng: e.latLng.lng() };
      });

      ready.value = true;
      emit("maploaded", currentPosition.value);
    });

    onUnmounted(async () => {
      if (watcher) navigator.geolocation.clearWatch(watcher);
      if (clickListener) clickListener.remove();
    });

    return {
      mapDiv,
      currentPosition,
      searchMap,
      searchInput,
      ready,
      mapVisible,
    };
  },
};
</script>

<style lang="scss" scoped>
.search-wrapper {
  display: flex;
  width: 100%;
  justify-content: center;

  input {
    height: 40px;
    background: rgba(0, 0, 0, 0.2);
    border-radius: 2px;
    outline: 0;
    border: 1px solid #eee;
    color: #fff;
    font-size: 17px;
    padding: 0 5px;
    width: 50%;
  }

  button {
    border-radius: 0;
    border: 0;
    padding: 0 20px;
  }
}

.map-wrapper {
  text-align: center;
  width: 100%;
  margin-top: 20px;
  margin-bottom: 20px;

  > div:nth-child(1) {
    margin-top: 5px;
    margin-bottom: 20px;
    text-decoration: underline;
    cursor: pointer;
  }
  .the-map {
    border-radius: 10px;
  }
}
</style>