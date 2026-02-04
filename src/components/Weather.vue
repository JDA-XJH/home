<template>
  <div class="weather" v-if="weatherData.city && weatherData.data.type">
    <span class="city-name">{{ simplifyCity(weatherData.city) }}</span>
    <span class="weather-type">&nbsp;·&nbsp;{{ weatherData.data.type }}</span>
    <span class="temp">&nbsp;{{ weatherData.data.temp }}°</span>
    <span class="sm-hidden wind-info">
      &nbsp;|&nbsp;{{ simplifyWind(weatherData.data.fengxiang) }}{{ weatherData.data.fengli }}
    </span>
  </div>
  <div class="weather-error" v-else>
    <span>Weather Hidden</span>
  </div>
</template>

<script setup>
import { reactive, onMounted, h } from "vue";
import { Error } from "@icon-park/vue-next";
import { ElMessage } from "element-plus";
import axios from "axios";

// 1. 响应式数据
const weatherData = reactive({
  city: null,
  data: {
    type: null,
    temp: null,
    fengxiang: null,
    fengli: null,
  },
});

// 2. 辅助函数：缩短城市名（国内去掉市区县，国外取前段）
const simplifyCity = (name) => {
  if (!name) return "Unknown";
  // 针对中文处理
  let city = name.replace(/[市区县省]$/g, "");
  // 针对国外城市名（如 San Francisco, CA）只取逗号前的部分
  city = city.split(",")[0];
  // 最终截取前 6 个字符防止撑爆
  return city.length > 6 ? city.substring(0, 6) + ".." : city;
};

// 3. 辅助函数：缩短风向
const simplifyWind = (dir) => {
  if (!dir) return "";
  const map = {
    "北风": "Norte", "东北风": "NE", "东风": "E", "东南风": "SE",
    "南风": "S", "西南风": "SW", "西风": "W", "西北风": "NW",
    "North": "N", "Northeast": "NE", "East": "E", "Southeast": "SE",
    "South": "S", "Southwest": "SW", "West": "W", "Northwest": "NW"
  };
  return map[dir] || dir;
};

// WMO 天气代码转西语描述（由Gemini翻译）
const weatherMap = {
  0: "Despejado",         // 晴朗
  1: "Parcialmente nublado", // 晴间多云
  2: "Nublado",           // 多云
  3: "Cubierto",          // 阴天 (更强调云层覆盖)
  45: "Niebla",           // 雾
  48: "Calima",           // 霾 (也可以用 Neblina 描述薄雾)
  51: "Llovizna",         // 毛毛雨
  61: "Lluvia ligera",    // 小雨
  71: "Nieve ligera",     // 小雪
  80: "Chubascos",        // 阵雨
  95: "Tormenta"          // 雷阵雨
};

// Obtener escala de viento (Escala de Beaufort) 获取风力等级 (蒲福氏风级) [Traducido con Gemini]
const getWindScale = (speed) => {
  if (speed < 1) return "Calma";            // 0级：无风/平静
  if (speed < 6) return "Ventolina";        // 1级：软风
  if (speed < 12) return "Brisa muy débil"; // 2级：轻风
  if (speed < 20) return "Brisa ligera";    // 3级：微风
  if (speed < 29) return "Brisa moderada";  // 4级：和风
  if (speed < 39) return "Brisa fresca";    // 5级：清风
  return "Viento fuerte";                   // 强风 (6级及以上)
};

// 获取风向描述
const getWindDir = (deg) => {
  const dirs = ["北", "东北", "东", "东南", "南", "西南", "西", "西北"];
  return dirs[Math.round(deg / 45) % 8] + "风";
};

const getWeatherData = async () => {
  try {
    // 1. 获取地理位置 (ipapi.co 国内国外访问均较稳定)
    const locRes = await axios.get("https://ipapi.co/json/");
    const { latitude, longitude, city } = locRes.data;
    
    // 如果获取不到城市名，用 "Unknown" 垫底，防止模板报错
    weatherData.city = city || "Somewhere";

    // 2. 获取天气 (Open-Meteo)
    const weatherUrl = `https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&current_weather=true`;
    const res = await axios.get(weatherUrl);
    
    if (res.data && res.data.current_weather) {
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
    // 如果 API 挂了，给一个默认显示状态，不让页面空白
    weatherData.city = "Unknown";
    weatherData.data.type = "Error"; 
    onError("No se pueden obtener datos meteorológicos.");
  }
};

const onError = (message) => {
  ElMessage({
    message,
    icon: h(Error, { theme: "filled", fill: "#efefef" }),
  });
};

onMounted(() => {
  getWeatherData();
});
</script>

<style scoped>
/* 建议加上一些基础样式防止挤在一起 */
.weather {
  display: flex;
  align-items: center;
  white-space: nowrap;
}
.city-name {
  font-weight: bold;
}
</style>