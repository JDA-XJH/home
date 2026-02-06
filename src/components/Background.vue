<template>
  <div :class="store.backgroundShow ? 'cover show' : 'cover'">
    <img
      v-show="store.imgLoadStatus"
      :src="bgUrl"
      class="bg"
      alt="cover"
      @load="imgLoadComplete"
      @error.once="imgLoadError"
      @animationend="imgAnimationEnd"
    />
    <div :class="store.backgroundShow ? 'gray hidden' : 'gray'" />
    <Transition name="fade" mode="out-in">
      <a
        v-if="store.backgroundShow && store.coverType != '3'"
        class="down"
        :href="bgUrl"
        target="_blank"
      >
        Descargar fondo de pantalla
      </a>
    </Transition>
  </div>
</template>

<script setup>
import { ref, watch, onMounted, onBeforeUnmount, h } from "vue";
import { mainStore } from "@/store";
import { Error } from "@icon-park/vue-next";

const store = mainStore();
const bgUrl = ref("");
const imgTimeout = ref(null);
const emit = defineEmits(["loadComplete"]);

// 获取环境变量
const mainUrl = import.meta.env.VITE_EXTERNAL_BACKGROUND_URL;
// 壁纸随机数 (1-12)
const bgRandom = Math.floor(Math.random() * 12 + 1);

/**
 * 壁纸切换逻辑
 * @param {string|number} type 切换类型
 */
const changeBg = (type) => {
  if (type == 0) {
    // 默认壁纸
    if (mainUrl) {
      bgUrl.value = mainUrl.replace('{bgrandom}', bgRandom);
    } else {
      bgUrl.value = `/home-image/background${bgRandom}.jpg`;
    }
  } else if (type == 1) {
    // 必应每日
    bgUrl.value = "https://bing.img.run/uhd.php";
  } else if (type == 2) {
    // 必应随机
    bgUrl.value = "https://bing.img.run/rand_uhd.php";
  } else if (type == 3) {
    // 随机风景
    bgUrl.value = "https://api.timelessq.com/bing/random";
  }
};

// 图片加载完成
const imgLoadComplete = () => {
  imgTimeout.value = setTimeout(
    () => {
      store.setImgLoadStatus(true);
    },
    Math.floor(Math.random() * (600 - 300 + 1)) + 300,
  );
};

// 图片动画完成
const imgAnimationEnd = () => {
  emit("loadComplete");
};

// 图片显示失败回退
const imgLoadError = () => {
  console.error("Fondo de pantalla fallido:", bgUrl.value);
  if (typeof ElMessage !== 'undefined') {
    ElMessage({
      message: "Error al cargar el fondo, usando imagen predeterminada.",
      icon: h(Error, {
        theme: "filled",
        fill: "#efefef",
      }),
    });
  }
  bgUrl.value = `/home-image/background${bgRandom}.jpg`;
};

// 监听类型变化
watch(
  () => store.coverType,
  (value) => {
    store.setImgLoadStatus(false);
    changeBg(value);
  },
);

onMounted(() => {
  changeBg(store.coverType);
});

onBeforeUnmount(() => {
  if (imgTimeout.value) clearTimeout(imgTimeout.value);
});
</script>

<style lang="scss" scoped>
.cover {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  transition: 0.25s;
  z-index: -1;

  &.show {
    z-index: 1;
  }

  .bg {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
    backface-visibility: hidden;
    filter: blur(20px) brightness(0.3);
    transition: filter 0.3s, transform 0.3s;
    animation: fade-blur-in 0.8s cubic-bezier(0.25, 0.46, 0.45, 0.94) forwards;
    animation-delay: 0.45s;
  }

  .gray {
    opacity: 1;
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background-image: radial-gradient(rgba(0, 0, 0, 0) 0, rgba(0, 0, 0, 0.5) 100%),
      radial-gradient(rgba(0, 0, 0, 0) 33%, rgba(0, 0, 0, 0.3) 166%);
    transition: 1.5s;
    &.hidden {
      opacity: 0;
    }
  }

  .down {
    font-size: 14px;
    color: white;
    position: absolute;
    bottom: 30px;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 12px 24px;
    border-radius: 8px;
    background-color: rgba(0, 0, 0, 0.3);
    min-width: 200px;
    text-decoration: none;
    backdrop-filter: blur(4px);
    transition: all 0.3s;

    &:hover {
      transform: translateX(-50%) scale(1.05);
      background-color: rgba(0, 0, 0, 0.6);
    }
    &:active {
      transform: translateX(-50%) scale(0.95);
    }
  }
}

@keyframes fade-blur-in {
  from {
    filter: blur(50px) brightness(0);
    transform: scale(1.1);
  }
  to {
    filter: blur(20px) brightness(0.3);
    transform: scale(1);
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>