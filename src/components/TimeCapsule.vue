<template>
  <div class="time-capsule">
    <div class="title">
      <hourglass-full theme="two-tone" size="24" :fill="['#efefef', '#00000020']" />
      <span>Cápsula del tiempo</span>
    </div>
    <span class="text">Hoy ya pasó&nbsp;{{ timeData.day.elapsed }}&nbsp;horas</span>
    <el-progress :text-inside="true" :stroke-width="20" :percentage="timeData.day.pass" />
    <span class="text">Esta semana ya pasó.&nbsp;{{ timeData.week.elapsed }}&nbsp;días</span>
    <el-progress :text-inside="true" :stroke-width="20" :percentage="timeData.week.pass" />
    <span class="text">Este mes ya pasó&nbsp;{{ timeData.month.elapsed }}&nbsp;días</span>
    <el-progress :text-inside="true" :stroke-width="20" :percentage="timeData.month.pass" />
    <span class="text">Este año ya pasó.&nbsp;{{ timeData.year.elapsed }}&nbsp;meses</span>
    <el-progress :text-inside="true" :stroke-width="20" :percentage="timeData.year.pass" />
    <div v-if="startDate?.length >= 4 && store.siteStartShow">
      <span class="text" v-html="startDateText" />
      <!-- <el-progress
        :show-text="false"
        :indeterminate="true"
        :stroke-width="6"
        :percentage="80"
        :duration="2"
      /> -->
    </div>
  </div>
</template>

<script setup>
import { HourglassFull } from "@icon-park/vue-next";
import { getTimeCapsule, siteDateStatistics } from "@/utils/getTime.js";
import { mainStore } from "@/store";
const store = mainStore();

// 进度条数据
const timeData = ref(getTimeCapsule());
const startDate = ref(import.meta.env.VITE_SITE_START);
const startDateText = ref(null);
const timeInterval = ref(null);

onMounted(() => {
  timeInterval.value = setInterval(() => {
    timeData.value = getTimeCapsule();
    if (startDate.value) startDateText.value = siteDateStatistics(new Date(startDate.value));
  }, 1000);
});

onBeforeUnmount(() => {
  clearInterval(timeInterval.value);
});
</script>

<style lang="scss" scoped>
.time-capsule {
  width: 100%;
  .title {
    display: flex;
    flex-direction: row;
    align-items: center;
    margin: 0.2rem 0 1.5rem;
    font-size: 1.1rem;
    .i-icon {
      display: flex;
      justify-content: center;
      align-items: center;
      margin-right: 6px;
    }
  }
  .text {
    display: block;
    margin: 1rem 0rem 0.5rem 0rem;
    font-size: 0.95rem;
  }
}
</style>
