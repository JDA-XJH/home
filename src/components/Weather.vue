<template>
  <div class="weather-wrapper" v-if="weatherData.city && weatherData.data.type">
    <span class="city-name">{{ weatherData.city }}</span>
    
    <span class="divider">|</span>

    <div class="carousel-container">
      <transition name="slide-fade" mode="out-in">
        
        <div v-if="step === 0" :key="0" class="weather-item">
          <span class="index-tag">TEMP</span>
          <span class="weather-icon">{{ extractEmoji(weatherData.data.type) }}</span>
          <span class="temp" :style="{ color: getTempColor(weatherData.data.temp) }">
            {{ weatherData.data.temp }}°C
          </span>
        </div>

        <div v-else-if="step === 1" :key="1" class="weather-item">
          <span class="index-tag">COND</span>
          <span class="weather-type">{{ weatherData.data.type }}</span>
        </div>

        <div v-else :key="2" class="weather-item">
          <span class="index-tag">VIENTO</span>
          <span class="wind-full">
            {{ weatherData.data.fengxiang }} {{ weatherData.data.windSpeed }} km/h
          </span>
        </div>

      </transition>
    </div>
  </div>
  <div class="weather-error" v-else>
    <span>Cargando clima...</span>
  </div>
</template>

<script setup>
import { reactive, onMounted, onUnmounted, ref } from "vue";
import axios from "axios";

const weatherData = reactive({
  city: null,
  data: { type: null, temp: null, fengxiang: null, windSpeed: null },
});

const step = ref(0);
let timer = null;

const getTempColor = (temp) => {
  if (temp <= 5) return "#00bfff";
  if (temp <= 15) return "#50c878";
  if (temp <= 28) return "#ffcc00";
  return "#ff4500";
};

const extractEmoji = (type) => type?.match(/[\uD800-\uDBFF][\uDC00-\uDFFF]|\u2600|\u2601|\u26C5/)?.[0] || "🌡️";

const weatherMap = {
  0: "☀️ Despejado", 1: "🌤️ Parcial", 2: "⛅️ Nublado", 3: "☁️ Cubierto",
  45: "🌫 Niebla", 48: "🌫 Calima", 51: "🌧️ Llovizna", 61: "🌧 Ligera",
  71: "🌨 Ligera", 80: "🌧️ Chubascos", 95: "⛈️ Tormenta"
};

const getWindDirLabel = (deg) => {
  const dirs = ["Norte", "Noreste", "Este", "Sureste", "Sur", "Suroeste", "Oeste", "Noroeste"];
  return dirs[Math.round(deg / 45) % 8];
};

const getWeatherData = async () => {
  try {
    const locRes = await axios.get("https://ipapi.co/json/");
    weatherData.city = locRes.data.city || "Desconocido";

    const res = await axios.get(`https://api.open-meteo.com/v1/forecast?latitude=${locRes.data.latitude}&longitude=${locRes.data.longitude}&current_weather=true`);
    
    if (res.data?.current_weather) {
      const now = res.data.current_weather;
      weatherData.data = {
        type: weatherMap[now.weathercode] || "Desconocido",
        temp: Math.round(now.temperature),
        fengxiang: getWindDirLabel(now.winddirection),
        windSpeed: now.windspeed,
      };
    }
  } catch (error) {
    console.error("Error fetching weather:", error);
  }
};

onMounted(() => {
  getWeatherData();
  timer = setInterval(() => { step.value = (step.value + 1) % 3; }, 3500);
});

onUnmounted(() => clearInterval(timer));
</script>

<style scoped>
.weather-wrapper {
  display: inline-flex;
  height: 30px;
  align-items: center;
  padding: 0 12px;
  color: #fff;
  border-radius: 4px;
  font-family: 'Segoe UI', system-ui, sans-serif;
  /* background: rgba(255, 255, 255, 0.05); 如果背景太单调可以开启这行 */
}

.city-name {
  font-size: 14px;
  font-weight: 700;
  color: #ffffff;
  /* 确保城市名不会因为后面文字长短而跳动 */
  min-width: fit-content;
}

.divider {
  margin: 0 12px;
  color: #444;
  font-weight: 200;
  user-select: none;
}

.carousel-container {
  display: inline-block;
  min-width: 120px; /* 给轮播区一个最小宽度，防止切换时整体抖动 */
}

.index-tag {
  font-size: 9px;
  background: #007aff; 
  color: #fff;
  padding: 1px 5px;
  border-radius: 3px;
  margin-right: 8px;
  font-weight: 800;
  letter-spacing: 0.5px;
}

.weather-item {
  display: flex;
  align-items: center;
  white-space: nowrap;
}

.weather-icon {
  margin-right: 4px;
}

.temp { font-weight: 700; font-size: 15px; }
.weather-type, .wind-full {
  font-size: 13px;
  color: #efefef;
  font-weight: 500;
}

/* 动画效果：上下滑动 */
.slide-fade-enter-active, .slide-fade-leave-active {
  transition: all 0.4s ease;
}
.slide-fade-enter-from { transform: translateY(8px); opacity: 0; }
.slide-fade-leave-to { transform: translateY(-8px); opacity: 0; }
</style>