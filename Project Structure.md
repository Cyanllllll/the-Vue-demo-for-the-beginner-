# 项目描述
整个项目分为前后端两部分，作者的工作主要为前端展示后台的实验数据和功能可视化
# 项目结构
## App.vue
```
<template>
  <div class="layout">
    <TopBar />//顶部深色
    <div class="body">
      <SideNav v-model:collapsed="collapsed" />//侧边栏
     <main class="main">
        <RouterView />//右侧路由视图
      </main>
    </div>
    <!-- <LogDrawer v-model:show="showLog" /> -->
 
  </div>
</template>
```
## Router/index.js
主要记录对有侧视图组件的路由
```
const routes = [
  { path: '/',                 name: '总览',     component: () => import('@/views/Overview.vue') },
  { path: '/gesture-recognition',     name: '手势识别', component: () => import('@/views/GestureRecognition.vue') },
  { path: '/finger-tracing',      name: '手指追踪', component: () => import('@/views/FingerTracing.vue') },
  { path: '/finger-accuracy',     name: '手指追踪精度', component: () => import('@/views/FingerAccuracy.vue') },
  { path: '/finger-latency',     name: '手指追踪延迟', component: () => import('@/views/FingerLatency.vue') },
  { path: '/eye-tracing',      name: '眼动追踪', component: () => import('@/views/EyeTracing.vue') },
  { path: '/eye-accuracy',     name: '眼动追踪精度', component: () => import('@/views/EyeAccuracy.vue') },
  { path: '/eye-hardware',     name: '眼动效率指标', component: () => import('@/views/EyeHardware.vue') },
  { path: '/hand-data-acquisition',     name: '手势数据采集', component: () => import('@/views/HandDataAcquisition.vue') },
  { path: '/eye-data-acquisition',     name: '眼动数据采集', component: () => import('@/views/EyeDataAcquisition.vue') },
  { path: '/data-alignment',             name: '数据对齐', component: () => import('@/views/DataAlignment.vue') },
  { path: '/data-fusion',     name: '数据融合', component: () => import('@/views/DataFusion.vue') },
  { path: '/interaction',     name: '个性化交互', component: () => import('@/views/Interaction.vue') },
  { path: '/crossdomain-eye-tracing',     name: '跨域眼动追踪', component: () => import('@/views/CrossDomainEyeTracing.vue') },
]
```
