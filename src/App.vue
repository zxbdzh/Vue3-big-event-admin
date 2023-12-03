<script setup>
import { useRouter } from 'vue-router'
import { useUserStore, useCountStore } from '@/stores'

const router = useRouter()

const goList = () => {
  // 在Vue3 compossitionAPI中
  // 1. 获取路由对象 router useRouter
  // 2. 获取路由参数 route useRoute
  router.push('/list')
}

const userStore = useUserStore()
const countStore = useCountStore()
</script>

<template>
  <div>
    <hr />
    <router-view></router-view>
    <br />
    <br />
    <br />
    <br />
    <br />
    <br />
    <br />
    <br />

    我是App
    <test-demo></test-demo>
    <el-button @click="$router.push('/home')">跳首页</el-button>
    <el-button @click="goList">跳列表页</el-button>

    <el-button type="primary">Primary</el-button>
    <el-button type="success">Success</el-button>

    <p>{{ userStore.token }}</p>

    <el-button @click="userStore.setToken('ASDFGHJKL;')">登录</el-button>
    <el-button @click="userStore.removeToken">退出</el-button>
    <hr />

    {{ countStore.count }}
    <el-button @click="countStore.add(2)">加法2</el-button>
  </div>
</template>

<style scoped></style>
