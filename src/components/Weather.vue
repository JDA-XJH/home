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
  return city.length > 10 ? city.substring(0, 10) + ".." : city;
};

// 3. 辅助函数：缩短风向
const simplifyWind = (dir) => {
  if (!dir) return "";
  const map = {
    "北风": "北", "东北风": "东北", "东风": "东", "东南风": "东南",
    "南风": "南", "西南风": "西南", "西风": "西", "西北风": "西北",
    "North": "N", "Northeast": "NE", "East": "E", "Southeast": "SE",
    "South": "S", "Southwest": "SW", "West": "W", "Northwest": "NW"
  };
  return map[dir] || dir;
};

// WMO 天气代码转中文描述
const weatherMap = {
  0: "晴朗", 1: "晴间多云", 2: "多云", 3: "阴天",
  45: "雾", 48: "霾", 51: "毛毛雨", 61: "小雨",
  71: "小雪", 80: "阵雨", 95: "雷阵雨"
};

// 获取风力等级 (蒲福氏风级)
const getWindScale = (speed) => {
  if (speed < 1) return "0级";
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
        type: weatherMap[now.weathercode] || "未知",
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