<template>
  <div class="weather" v-if="weatherData.city && weatherData.data.type">
    <span>{{ weatherData.city }}&nbsp;</span>
    <span>{{ weatherData.data.type }}&nbsp;</span>
    <span>{{ weatherData.data.temp }}°C</span>
    <span class="sm-hidden">
      &nbsp;{{ weatherData.data.fengxiang }}&nbsp;
    </span>
    <span class="sm-hidden">{{ weatherData.data.fengli }}</span>
  </div>
  <div class="weather" v-else>
    <span>Falló la adquisición de datos meteorológicos</span>
  </div>
</template>

<script setup>
import { reactive, onMounted, h } from "vue";
import { Error } from "@icon-park/vue-next";
import { ElMessage } from "element-plus";
import axios from "axios";

const weatherData = reactive({
  city: null,
  data: {
    type: null,
    temp: null,
    fengxiang: null,
    fengli: null,
  },
});

// WMO 天气代码转中文描述
const weatherMap = {
  0: "晴朗", 1: "晴间多云", 2: "多云", 3: "阴天",
  45: "雾", 48: "霾", 51: "毛毛雨", 61: "小雨",
  71: "小雪", 80: "阵雨", 95: "雷阵雨"
};

// 获取风力等级 (蒲福氏风级)
const getWindScale = (speed) => {
  if (speed < 1) return "无风";
  if (speed < 6) return "1级";
  if (speed < 12) return "2级";
  if (speed < 20) return "3级";
  if (speed < 29) return "4级";
  if (speed < 39) return "5级";
  return "强风";
};

// 获取风向描述
const getWindDir = (deg) => {
  const dirs = ["北", "东北", "东", "东南", "南", "西南", "西", "西北"];
  return dirs[Math.round(deg / 45) % 8] + "风";
};

const getWeatherData = async () => {
  try {
    // 1. 获取地理位置 (基于 IP，全球通用)
    const locRes = await axios.get("https://ipapi.co/json/");
    const { latitude, longitude, city } = locRes.data;
    weatherData.city = city;

    // 2. 获取天气 (Open-Meteo 全球免费接口)
    const weatherUrl = `https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&current_weather=true`;
    const res = await axios.get(weatherUrl);
    
    if (res.data.current_weather) {
      const now = res.data.current_weather;
      weatherData.data = {
        type: weatherMap[now.weathercode] || "未知",
        temp: Math.round(now.temperature),
        fengxiang: getWindDir(now.winddirection),
        fengli: getWindScale(now.windspeed),
      };
    }
  } catch (error) {
    console.error("Weather Fetch Error:", error);
    onError("无法获取天气数据");
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