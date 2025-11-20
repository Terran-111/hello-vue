<script setup>
  import { ref ,onMounted } from 'vue' // 引入 Vue 的核心魔法棒
  import MyButton from './components/MyButton.vue'

  // 定义一个“会动的变量” (响应式变量)
  // ref(0) 意思是初始值是 0
  const count = ref(0)
  const name = ref("Vue新手 ")

  const logs = ref([])

  // 1.新增：变量存储图片地址
   const catImage = ref('')

   // 2.新增：“抓猫”函数
   async function getCat(){
    try{
      const response = await fetch('https://cataas.com/cat?json=true')
      const data = await response.json()

      // catImage.value='https://cataas.com/cat/'+data._id
      // 直接使用API返回完整的URL
      catImage.value=data.url
    
      logs.value.push('抓到了一只新猫猫！')
    } catch(e){
      console.log('抓猫失败：',e)
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