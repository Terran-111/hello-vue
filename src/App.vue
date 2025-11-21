<script setup>
  import { ref ,onMounted,nextTick } from 'vue' // 引入 Vue 的核心魔法棒

  const count = ref(0)
  const name = ref("Vue全栈 ")
  const logs = ref([])
  const catImage = ref('')
  const isLoading = ref(false)

  const chatInput=ref('')
  const isChatting =ref(false)
  const chatHistory = ref([
    { role: 'ai', content: '喵？我是你的 AI 助手，有什么想聊的吗？' }
  ])
  // 用于自动滚动到底部
  const chatBoxRef = ref(null)

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

  // --- 新增：AI 聊天功能 ---
  async function sendMessage(){
    if (!chatInput.value.trim()) return

    const userMsg = chatInput.value
    chatInput.value=''
    isChatting.value=true

    // 添加用户消息
    chatHistory.value.push({
      role: 'user',
      content: userMsg
    })
    // 添加AI消息（初始为空）
    const aiMsgIndex = chatHistory.value.length
    chatHistory.value.push({
      role: 'assistant',
      content: ''
    })
    try{
      // 发给后端
      const response = await fetch('https://api.liberflux.top/chat', {
        method:'POST',
        headers:{'Content-Type':'application/json'},
        body:JSON.stringify({message:userMsg})
      })
      if (!response.ok) {
        throw new Error('网络请求失败')
      }
      // 处理流式响应
      const reader = response.body.getReader()
      const decoder = new TextDecoder()

      // 循环读取数据流
      while(true){
        const {done,value}=await reader.read()
        if (done) break
        // 解码并追加到对话框中
        const text = decoder.decode(value)
        chatHistory.value[aiMsgIndex].content += text
        scrollToBottom() // 有新字就滚到底部
      }
    } catch (e) {
      chatHistory.value[aiMsgIndex].content = '喵？出错了，请重试喵~'
    } finally{
      isChatting.value = false
      scrollToBottom()
    }
  }

  // 辅助函数：让聊天框自动滚到底部
  function scrollToBottom(){
    nextTick(()=>{
      if(chatBoxRef.value){
        chatBoxRef.value.scrollTop = chatBoxRef.value.scrollHeight
      }
    })
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
          <span>🏆 全栈AI 助手喵</span>
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

      <el-divider />

      <div class="chat-section">
        <h4>💬 和 AI 聊两句</h4>
        
        <div class="chat-window" ref="chatBoxRef">
          <div 
            v-for="(msg, index) in chatHistory" 
            :key="index"
            class="message-row"
            :class="msg.role === 'user' ? 'my-msg' : 'ai-msg'"
          >
            <div class="bubble">
              {{ msg.content }}
            </div>
          </div>
        </div>

        <div style="display: flex; gap: 10px; margin-top: 10px;">
          <el-input 
            v-model="chatInput" 
            placeholder="问我任何问题..." 
            @keyup.enter="sendMessage"
            :disabled="isChatting"
          />
          <el-button type="primary" @click="sendMessage" :loading="isChatting">
            发送
          </el-button>
        </div>
      </div>

      <el-divider /> 

      <div style="text-align: center;">
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

/* 聊天窗口样式 */
.chat-window {
  height: 200px;
  border: 1px solid #eee;
  border-radius: 8px;
  padding: 10px;
  overflow-y: auto;
  background: #fff;
  margin-bottom: 10px;
}
.message-row {
  display: flex;
  margin-bottom: 10px;
}
.my-msg {
  justify-content: flex-end; /* 我发的消息靠右 */
}
.ai-msg {
  justify-content: flex-start; /* AI发的消息靠左 */
}
.bubble {
  max-width: 80%;
  padding: 8px 12px;
  border-radius: 12px;
  font-size: 14px;
  line-height: 1.5;
}
.my-msg .bubble {
  background-color: #409eff;
  color: white;
  border-bottom-right-radius: 2px;
}
.ai-msg .bubble {
  background-color: #f4f4f5;
  color: #333;
  border-bottom-left-radius: 2px;
}
</style>