<script setup>
  import { ref ,onMounted } from 'vue' // 引入 Vue 的核心魔法棒

  // 定义一个“会动的变量” (响应式变量)
  // ref(0) 意思是初始值是 0
  const count = ref(0)
  const name = ref("Vue新手 ")

  const logs = ref([])

  const isLoading = ref(false)
  // 1.新增：变量存储图片地址
   const catImage = ref('')

   // 2.新增：“抓猫”函数
   async function getCat(){
    isLoading.value = true
    try{
      // const response = await fetch('https://cataas.com/cat?json=true')

      // 以前是找cataas.com的API，现在我们用自家兄弟Python (8000端口)
      // const response = await fetch('http://127.0.0.1:8000/cat?json=true')

      // 现在找云端的API,https://my-python-backend-你的名字.vercel.app
      //const response = await fetch('https://my-python-backend-wine.vercel.app/cat?json=true')

      // 现在使用自己的域名，让其他人不用挂梯子也可以访问
      const response = await fetch('https://api.liberflux.top/cat?json=true')
      const data = await response.json()

      // catImage.value='https://cataas.com/cat/'+data._id
      // 直接使用API返回完整的URL
      // catImage.value=data.url
      catImage.value=data.image
    
      logs.value.push(data.note)

    } catch (e) {
      console.log("出错了", e)
      logs.value.push("连不上后端，请检查 Python 跑起来没？")
    } finally {
      isLoading.value = false
    }
  }

  // 定义一个函数：点击后执行什么
  function add() {
    count.value = count.value + 1
    console.log("现在的数字是：", count.value)
    logs.value.push("你点击了按钮，数字变成了"+count.value)
  }

  function reset(){
    count.value=0
    logs.value=[]
  }

  // 使用 onMounted() ,页面加载完成，自动执行一些初始化操作
  onMounted(()=>{
    console.log('网页加载完毕，自动抓第一只猫...')
    getCat()
  })
</script>

<template>
  <div class="app-container">
    
    <el-card class="box-card" style="max-width: 480px; margin: 0 auto;">
      <template #header>
        <div class="card-header">
          <span>🏆 Vue 进阶练习</span>
        </div>
      </template>

      <div style="margin-bottom: 20px;">
        <el-input 
          v-model="name" 
          placeholder="请输入你的大名" 
          clearable
          size="large"
        >
          <template #prepend>用户</template>
        </el-input>
      </div>

      <div style="text-align: center;">
        <h1>Hello, {{ name }}</h1>
        <p class="number" :style="{ color: count > 10 ? '#f56c6c' : '#409eff' }">
          {{ count }}
        </p>
        
        <el-button type="primary" size="large" @click="add" round>点我 +1</el-button>
        <el-button type="danger" size="large" @click="reset" v-if="count > 0" circle>重置</el-button>
      </div>

      <el-divider /> <div style="text-align: center;">
        <h4>🐱 每日吸猫</h4>
        <el-image 
          style="width: 200px; height: 200px; border-radius: 8px;"
          :src="catImage" 
          :preview-src-list="[catImage]"
          fit="cover"
        >
          <template #error>
            <div class="image-slot">😿 加载失败</div>
          </template>
        </el-image>
        <br><br>
        <el-button type="success" @click="getCat" :loading="isLoading" round>
          {{ isLoading ? '抓取中...' : '换一只猫' }}
        </el-button>
      </div>

      <el-divider content-position="left">操作日志</el-divider>

      <div style="height: 200px; overflow-y: auto;">
        <el-timeline>
          <el-timeline-item
            v-for="(item, index) in logs"
            :key="index"
            :type="index === logs.length - 1 ? 'primary' : ''"
            :timestamp="'第 ' + (index + 1) + ' 次'"
          >
            {{ item }}
          </el-timeline-item>
        </el-timeline>
      </div>

    </el-card>
  </div>
</template>

<style scoped>
.app-container {
  padding: 40px;
  background-color: #f0f2f5; /* 浅灰色背景 */
  min-height: 100vh;
}

.number {
  font-size: 60px;
  font-weight: bold;
  margin: 10px 0;
  transition: color 0.3s;
}

/* 修复 image error 插槽的样式 */
.image-slot {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
  background: #f5f7fa;
  color: #909399;
}
</style>