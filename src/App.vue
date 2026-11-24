<template>
  <div style="padding: 20px; max-width: 900px; margin: 0 auto; font-family: sans-serif;">
    
    <!-- 如果没登录，显示登录框 -->
    <div v-if="!isLoggedIn" style="margin-top: 100px; text-align: center; border: 1px solid #ddd; padding: 40px; border-radius: 8px; max-width: 400px; margin-left: auto; margin-right: auto;">
      <h2>系统登录</h2>
      <div style="margin-bottom: 15px;">
        <input v-model="username" placeholder="账号 (admin)" style="padding: 8px; width: 80%;"/>
      </div>
      <div style="margin-bottom: 20px;">
        <input v-model="password" type="password" placeholder="密码 (123456)" style="padding: 8px; width: 80%;"/>
      </div>
      <button @click="handleLogin" style="padding: 10px 30px; cursor: pointer; background-color: #4CAF50; color: white; border: none; border-radius: 4px;">登录</button>
    </div>

    <!-- 如果已登录，显示设备管理界面 -->
    <div v-else>
      <div style="display: flex; justify-content: space-between; align-items: center;">
        <h2>设备资产管理系统</h2>
        <button @click="handleLogout" style="padding: 5px 15px; cursor: pointer;">退出登录</button>
      </div>

      <div style="border: 1px solid #ccc; padding: 15px; margin-bottom: 20px; border-radius: 5px; background-color: #f9f9f9;">
        <h3>新增设备</h3>
        <input v-model="form.name" placeholder="设备名称" style="margin-right: 10px; padding: 5px;"/>
        <input v-model="form.category" placeholder="分类" style="margin-right: 10px; padding: 5px;"/>
        <input v-model="form.status" placeholder="状态" style="margin-right: 10px; padding: 5px;"/>
        <button @click="addDevice" style="padding: 5px 15px; cursor: pointer;">添加</button>
      </div>

      <button @click="fetchDevices" style="padding: 8px 16px; cursor: pointer; margin-bottom: 10px;">刷新设备列表</button>
      
      <table border="1" style="width: 100%; border-collapse: collapse; text-align: left;">
        <thead style="background-color: #f2f2f2;">
          <tr>
            <th style="padding: 10px;">ID</th>
            <th style="padding: 10px;">设备名称</th>
            <th style="padding: 10px;">分类</th>
            <th style="padding: 10px;">状态</th>
            <th style="padding: 10px;">操作</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="device in devices" :key="device.id">
            <td style="padding: 10px;">{{ device.id }}</td>
            <td style="padding: 10px;">{{ device.name }}</td>
            <td style="padding: 10px;">{{ device.category }}</td>
            <td style="padding: 10px;">{{ device.status }}</td>
            <td style="padding: 10px;">
              <button @click="deleteDevice(device.id)" style="color: red; cursor: pointer;">删除</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

// --- 状态变量 ---
const isLoggedIn = ref(false) // 是否已登录
const username = ref('')
const password = ref('')
const devices = ref([])
const form = ref({ name: '', category: '', status: '正常' })

// --- 登录逻辑 ---
const handleLogin = async () => {
  try {
    const res = await axios.post('http://localhost:8080/api/login', {
      username: username.value,
      password: password.value
    })
    if (res.data.code === 200) {
      localStorage.setItem('token', res.data.token) // 把 token 存到浏览器本地
      isLoggedIn.value = true // 切换页面状态
      fetchDevices() // 登录成功后拉取数据
    } else {
      alert(res.data.msg)
    }
  } catch (error) {
    alert('登录失败，请检查后端是否启动')
  }
}

// --- 退出登录 ---
const handleLogout = () => {
  localStorage.removeItem('token')
  isLoggedIn.value = false
  devices.value = []
}

// --- 封装带 Token 的请求头 ---
const getAuthHeader = () => {
  const token = localStorage.getItem('token')
  return { headers: { 'Authorization': token } }
}

// --- 获取设备列表 ---
const fetchDevices = async () => {
  try {
    const res = await axios.get('http://localhost:8080/api/devices', getAuthHeader())
    devices.value = res.data
  } catch (error) {
    if (error.response && error.response.status === 401) {
      alert('登录过期，请重新登录')
      handleLogout()
    }
  }
}

// --- 新增设备 ---
const addDevice = async () => {
  if (!form.value.name) return alert('请输入设备名称')
  await axios.post('http://localhost:8080/api/devices', form.value, getAuthHeader())
  form.value = { name: '', category: '', status: '正常' }
  fetchDevices()
}

// --- 删除设备 ---
const deleteDevice = async (id) => {
  if (confirm('确定要删除吗？')) {
    await axios.delete(`http://localhost:8080/api/devices/${id}`, getAuthHeader())
    fetchDevices()
  }
}

// 页面加载时，检查本地有没有 Token
onMounted(() => {
  const token = localStorage.getItem('token')
  if (token) {
    isLoggedIn.value = true
    fetchDevices()
  }
})
</script>