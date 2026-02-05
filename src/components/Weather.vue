<template>
  <div class="weather-wrapper" v-if="weatherData.city && weatherData.data.type">
    <div class="static-side">
      <span class="city-name">{{ weatherData.city }}</span>
      <span class="divider">|</span>
    </div>

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
          <span class="index-tag">🌀</span>
          <span class="wind-full">
            {{ weatherData.data.fengxiang }} | {{ weatherData.data.windSpeed }} km/h
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
    console.error("Error:", error);
  }
};

onMounted(() => {
  getWeatherData();
  timer = setInterval(() => { step.value = (step.value + 1) % 3; }, 4000);
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
  font-family: 'Segoe UI', system-ui, sans-serif;
  /* 防止外部容器塌陷 */
  overflow: hidden; 
}

/* 左侧固定：绝对不许动 */
.static-side {
  display: flex;
  align-items: center;
  flex-shrink: 0; /* 禁止被挤压 */
}

.city-name {
  font-size: 14px;
  font-weight: 700;
  color: #ffffff;
  white-space: nowrap;
}

.divider {
  margin: 0 10px;
  color: rgba(255, 255, 255, 0.2);
  user-select: none;
}

/* 右侧轮播：设定一个足够大的固定宽度 */
.carousel-container {
  position: relative;
  display: inline-flex;
  align-items: center;
  /* 这个宽度要根据你西语最长的那一行（通常是风向那行）来预估 */
  /* 如果你的城市空间非常紧，可以稍微调小这个值 */
  width: 160px; 
  height: 100%;
  flex-shrink: 0;
}

.weather-item {
  position: absolute; /* 使用绝对定位让各组重叠，切换时就不会撑开父容器 */
  left: 0;
  display: flex;
  align-items: center;
  white-space: nowrap;
}

.index-tag {
  font-size: 9px;
  background: #007aff; 
  color: #fff;
  padding: 1px 5px;
  border-radius: 3px;
  margin-right: 8px;
  font-weight: 800;
  flex-shrink: 0;
}

.temp { font-weight: 700; font-size: 15px; }
.weather-type, .wind-full {
  font-size: 13px;
  color: #efefef;
  font-weight: 500;
}

/* 动画效果 */
.slide-fade-enter-active, .slide-fade-leave-active {
  transition: all 0.5s ease;
}
.slide-fade-enter-from { transform: translateY(12px); opacity: 0; }
.slide-fade-leave-to { transform: translateY(-12px); opacity: 0; }
</style>