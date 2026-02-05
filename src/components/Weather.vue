<template>
  <div class="weather-wrapper" v-if="weatherData.city && weatherData.data.type">
    <transition name="slide-fade" mode="out-in">
      <div v-if="step === 0" :key="0" class="weather-item">
        <span class="city-name">{{ simplifyCity(weatherData.city) }}</span>
        <span class="divider">&nbsp;·&nbsp;</span>
        <span class="temp">{{ weatherData.data.temp }}°</span>
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
  <div class="weather-error" v-else>
    <span>Weather Hidden</span>
  </div>
</template>

<script setup>
import { reactive, onMounted, onUnmounted, ref, h } from "vue";
import { Error } from "@icon-park/vue-next";
import { ElMessage } from "element-plus";
import axios from "axios";

// 1. 响应式数据
const weatherData = reactive({
  city: null,
  data: { type: null, temp: null, fengxiang: null, fengli: null },
});

const step = ref(0); // 轮换步骤
let timer = null;

// 2. 城市缩写：Barcelona -> BCN
const simplifyCity = (name) => {
  if (!name) return "??";
  const cityMap = { "Barcelona": "BCN", "Madrid": "MAD", "Shanghai": "PVG", "Beijing": "PEK" };
  if (cityMap[name]) return cityMap[name];
  
  // 处理逻辑：如果是中文取前2字，如果是英文取逗号前的前3字母大写
  return /[\u4e00-\u9fa5]/.test(name) 
    ? name.replace(/[市区县省]$/g, "").substring(0, 2)
    : name.split(",")[0].substring(0, 3).toUpperCase();
};

// 3. 缩短风向图标
const simplifyWind = (dir) => {
  const map = {
    "北风": "↑", "东北风": "↗", "东风": "→", "东南风": "↘",
    "南风": "↓", "西南风": "↙", "西风": "←", "西北风": "↖"
  };
  return map[dir] || dir;
};

// WMO 天气映射
const weatherMap = {
  0: "☀️ Despejado", 1: "🌤️ Parcial", 2: "⛅️ Nublado", 3: "☁️ Cubierto",
  45: "🌫 Niebla", 48: "🌫 Calima", 51: "🌧️ Llovizna", 61: "🌧 Ligera",
  71: "🌨 Ligera", 80: "🌧️ Chubasco", 95: "⛈️ Tormenta"
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
    weatherData.city = city || "Unknown";

    const weatherUrl = `https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&current_weather=true`;
    const res = await axios.get(weatherUrl);
    
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
    console.error("Weather Fetch Error:", error);
    weatherData.city = "Unknown";
    onError("Error al obtener datos.");
  }
};

const onError = (message) => {
  ElMessage({ message, icon: h(Error, { theme: "filled", fill: "#efefef" }) });
};

onMounted(() => {
  getWeatherData();
  // 每 3 秒切换一次显示内容
  timer = setInterval(() => {
    step.value = (step.value + 1) % 3;
  }, 3000);
});

onUnmounted(() => {
  if (timer) clearInterval(timer);
});
</script>

<style scoped>
.weather-wrapper {
  display: inline-flex;
  height: 24px;
  align-items: center;
  overflow: hidden;
}

.weather-item {
  display: flex;
  align-items: center;
  white-space: nowrap;
  font-size: 13px;
  font-weight: 500;
}

.city-name {
  font-weight: 900;
  color: #409eff;
}

/* 动画：向上滑动淡出，向下滑动淡入 */
.slide-fade-enter-active, .slide-fade-leave-active {
  transition: all 0.5s ease;
}
.slide-fade-enter-from {
  transform: translateY(10px);
  opacity: 0;
}
.slide-fade-leave-to {
  transform: translateY(-10px);
  opacity: 0;
}
</style>