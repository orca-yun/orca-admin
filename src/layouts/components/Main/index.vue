<template>
  <Maximize v-show="maximize" />
  <Tabs v-show="tabs" />
  <el-main>
    <router-view v-slot="{ Component, route }">
      <transition appear name="fade-transform" mode="out-in">
        <div style="height: 100%">
          <Breadcrumb v-show="globalStore.breadcrumb" id="breadcrumb" />
          <keep-alive :include="keepAliveName">
            <component :is="Component" v-if="isRouterShow" :key="route.fullPath" />
          </keep-alive>
        </div>
      </transition>
    </router-view>
  </el-main>
  <!-- <el-footer v-show="footer">
    <Footer />
  </el-footer> -->
</template>

<script setup lang="ts">
import { ref, onBeforeUnmount, provide, watch } from "vue";
import { storeToRefs } from "pinia";
import { useDebounceFn } from "@vueuse/core";
import { useGlobalStore } from "@/stores/modules/global";
import { useKeepAliveStore } from "@/stores/modules/keepAlive";
import Maximize from "./components/Maximize.vue";
import Tabs from "@/layouts/components/Tabs/index.vue";
import Breadcrumb from "@/layouts/components/Header/components/Breadcrumb.vue";
// import Footer from "@/layouts/components/Footer/index.vue";

const globalStore = useGlobalStore();
const { maximize, isCollapse, layout, tabs } = storeToRefs(globalStore);

const keepAliveStore = useKeepAliveStore();
const { keepAliveName } = storeToRefs(keepAliveStore);

// 注入刷新页面方法
const isRouterShow = ref(true);
const refreshCurrentPage = (val: boolean) => (isRouterShow.value = val);
provide("refresh", refreshCurrentPage);

// 监听当前页面是否最大化，动态添加 class
watch(
  () => maximize.value,
  () => {
    const app = document.getElementById("app") as HTMLElement;
    if (maximize.value) app.classList.add("main-maximize");
    else app.classList.remove("main-maximize");
  },
  { immediate: true }
);

// 监听布局变化，在 body 上添加相对应的 layout class
watch(
  () => layout.value,
  () => {
    const body = document.body as HTMLElement;
    body.setAttribute("class", layout.value);
  },
  { immediate: true }
);

// 监听窗口大小变化，折叠侧边栏
const screenWidth = ref(0);
const listeningWindow = useDebounceFn(() => {
  screenWidth.value = document.body.clientWidth;
  if (!isCollapse.value && screenWidth.value < 1200) globalStore.setGlobalState("isCollapse", true);
  if (isCollapse.value && screenWidth.value > 1200) globalStore.setGlobalState("isCollapse", false);
}, 100);
window.addEventListener("resize", listeningWindow, false);
onBeforeUnmount(() => {
  window.removeEventListener("resize", listeningWindow);
});
</script>

<style scoped lang="scss">
@import "./index.scss";
#breadcrumb {
  padding: 10px 10px;
  background-color: #fff;
  font-family:
    PingFangSC,
    PingFang SC;
  font-weight: 600;
  font-size: 30px;
  color: #2e2f33;
  border-bottom: 1px solid #eff1f6;
  box-sizing: border-box;
}
</style>
