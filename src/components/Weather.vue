<template>
  <div class="weather-wrapper" v-if="weatherData.city && weatherData.data.type">
    <span class="city-name">{{ weatherData.city }}</span>
    <span class="divider">&nbsp;·&nbsp;</span>

    <div class="carousel-container">
      <transition name="slide-fade" mode="out-in">
        <div v-if="step === 0" :key="0" class="weather-item">
          <span class="weather-icon">{{ extractEmoji(weatherData.data.type) }}</span>
          <span class="temp" :style="{ color: getTempColor(weatherData.data.temp) }">
            {{ weatherData.data.temp }}°
          </span>
        </div>

        <div v-else-if="step === 1" :key="1" class="weather-item">
          <span class="weather-type">{{ weatherData.data.type }}</span>
        </div>

        <div v-else :key="2" class="weather-item">
          <span class="wind-info">
            {{ simplifyWind(weatherData.data.fengxiang) }} {{ weatherData.data.fengli }}
          </span>
        </div>
      </transition>
    </div>
  </div>
  <div class="weather-error" v-else>
    <span>Weather Loading...</span>
  </div>
</template>

<script setup>
import { reactive, onMounted, onUnmounted, ref, h } from "vue";
import { Error } from "@icon-park/vue-next";
import { ElMessage } from "element-plus";
import axios from "axios";

const weatherData = reactive({
  city: null,
  data: { type: null, temp: null, fengxiang: null, fengli: null },
});

const step = ref(0);
let timer = null;

// 1. 根据温度返回动态颜色
const getTempColor = (temp) => {
  if (temp <= 5) return "#00bfff"; // 极冷：深冰蓝
  if (temp <= 15) return "#50c878"; // 凉爽：翡翠绿
  if (temp <= 28) return "#ffcc00"; // 舒适：暖黄
  return "#ff4500"; // 炎热：橙红
};

// 2. 提取 Emoji
const extractEmoji = (type) => type?.match(/[\uD800-\uDBFF][\uDC00-\uDFFF]|\u2600|\u2601|\u26C5/)?.[0] || "🌡️";

// 3. 简化风向图标
const simplifyWind = (dir) => {
  const map = {
    "北风": "↑", "东北风": "↗", "东风": "→", "东南风": "↘",
    "南风": "↓", "西南风": "↙", "西风": "←", "西北风": "↖"
  };
  return map[dir] || dir;
};

const weatherMap = {
  0: "☀️ Despejado", 1: "🌤️ Parcial", 2: "⛅️ Nublado", 3: "☁️ Cubierto",
  45: "🌫 Niebla", 48: "🌫 Calima", 51: "🌧️ Llovizna", 61: "🌧 Ligera",
  71: "🌨 Ligera", 80: "🌧️ Chubascos", 95: "⛈️ Tormenta"
};

const getWindScale = (speed) => {
  if (speed < 1) return "Calma";
  if (speed < 12) return "Brisa";
  if (speed < 39) return "Moderado";
  return "Fuerte";
};

const getWindDir = (deg) => {
  const dirs = ["北风", "东北风", "东风", "东南风", "南风", "西南风", "西风", "西北风"];
  return dirs[Math.round(deg / 45) % 8];
};

const getWeatherData = async () => {
  try {
    const locRes = await axios.get("https://ipapi.co/json/");
    const { latitude, longitude, city } = locRes.data;
    weatherData.city = city || "Desconocido";

    const res = await axios.get(`https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&current_weather=true`);
    
    if (res.data?.current_weather) {
      const now = res.data.current_weather;
      weatherData.data = {
        type: weatherMap[now.weathercode] || "Unknown",
        temp: Math.round(now.temperature),
        fengxiang: getWindDir(now.winddirection),
        fengli: getWindScale(now.windspeed),
      };
    }
  } catch (error) {
    console.error("Weather Error:", error);
  }
};

onMounted(() => {
  getWeatherData();
  timer = setInterval(() => {
    step.value = (step.value + 1) % 3;
  }, 3000);
});

onUnmounted(() => clearInterval(timer));
</script>

<style scoped>
.weather-wrapper {
  display: inline-flex;
  height: 28px;
  align-items: center;
  padding: 0 10px;
  background: rgba(0, 0, 0, 0.05); /* 淡淡的底色提升质感 */
  border-radius: 6px;
  overflow: hidden;
  font-family: 'Segoe UI', Roboto, sans-serif;
}

.city-name {
  font-weight: 700;
  color: #444;
}

.divider {
  color: #999;
}

.carousel-container {
  display: inline-block;
}

.weather-item {
  display: flex;
  align-items: center;
  gap: 4px;
  white-space: nowrap;
  font-size: 14px;
  font-weight: 600;
}

.temp {
  transition: color 0.5s ease; /* 颜色变化更平滑 */
}

/* 动画效果 */
.slide-fade-enter-active, .slide-fade-leave-active {
  transition: all 0.5s ease;
}
.slide-fade-enter-from {
  transform: translateY(12px);
  opacity: 0;
}
.slide-fade-leave-to {
  transform: translateY(-12px);
  opacity: 0;
}
</style>