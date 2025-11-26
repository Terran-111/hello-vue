<script setup>
  import { ref ,onMounted,nextTick } from 'vue' 
  // 引入漂亮的图标
  import { User, Service, Refresh, Position, ChatLineRound, Trophy,Delete } from '@element-plus/icons-vue'
  import MarkdownIt from 'markdown-it' // 👈 新增
  // 1.基础数据
  const count = ref(0)
  const name = ref("")
  const logs = ref([])
  const catImage = ref('')
  const isLoading = ref(false)
  const isThinking = ref(false)
  // md
  const md = new MarkdownIt({
  linkify: true, // 允许识别网址自动变链接
  breaks: true   // 允许换行
})
  // 2.聊天数据
  const chatInput=ref('')
  const isChatting =ref(false)
  const chatHistory = ref([
    { role: 'assistant', content: '喵？我是你的 AI 助手，有什么想聊的吗？' }
  ])
  const chatBoxRef = ref(null)
  // 3.抓猫逻辑
  async function getCat(){
    isLoading.value = true
    try{
      const response = await fetch('https://api.liberflux.top/cat?json=true')
      const data = await response.json()

      catImage.value=data.image
      //logs.value.push(data.note)
      logs.value.unshift(data.note)// unshift 把日志加到最前面，方便看

    } catch (e) {
      console.log("出错了", e)
    } finally {
      isLoading.value = false
    }
  }
  // 👇 新增一个函数，专门用来在 HTML 里调用
  const renderMarkdown = (text) => {
    return md.render(text || '')
  }
  // --- AI 聊天功能（json模式） ---
  async function sendMessage(){
    if (!chatInput.value.trim()) return

    const userMsg = chatInput.value
    chatInput.value=''
    isChatting.value=true
    isThinking.value = true  

    // A. 先把自己说的话上屏
    chatHistory.value.push({
      role: 'user',
      content: userMsg
    })

    const historyToSend = chatHistory.value.slice(-20).map(msg => ({
      role: msg.role,
      content: msg.content.trim()
    }))
    // 添加AI消息（初始为空）
    const aiMsgIndex = chatHistory.value.length
    chatHistory.value.push({
      role: 'assistant',
      content: ''
    })
    
    scrollToBottom()
    try{
      // 发给后端
      const response = await fetch('https://api.liberflux.top/chat', {
        method:'POST',
        headers:{'Content-Type':'application/json'},
        body:JSON.stringify({history: historyToSend})
      })
      if (!response.ok) {
        throw new Error('网络请求失败')
      }

      // 处理流式响应
      const reader = response.body.getReader()
      const decoder = new TextDecoder()
      let isFirstChunk = true  // 标记是否是第一块数据
      // 循环读取数据流
      while(true){
        const {done,value}=await reader.read()
        if (done) break
        // 解码并追加到对话框中
        const text = decoder.decode(value)
        // 如果是第一块数据，结束思考状态
        if (isFirstChunk) {
          isThinking.value = false
          isFirstChunk = false
        }
        chatHistory.value[aiMsgIndex].content += text
        scrollToBottom() // 有新字就滚到底部
      }
    } catch (e) {
      isThinking.value = false  // 出错时也要结束思考状态
      chatHistory.value[aiMsgIndex].content = '喵呜...脑子短路了，请重试喵~'
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
    logs.value.unshift(`✨ 摸鱼能量 +1，当前：${count.value}`)
  }

  function reset(){
    count.value=0
    logs.value.unshift("🔄 能量归零啦")
  }

  // --- 新增：加载历史记录 ---
  async function loadHistory() {
    try {
      const res = await fetch('https://api.liberflux.top/history')
      const data = await res.json()
      
      // 如果数据库有数据，就覆盖默认的开场白
      if (data.history && data.history.length > 0) {
        // 数据库存的 role 是 'user'/'assistant'
        // 我们直接用就行，因为前端已经适配了 assistant
        chatHistory.value = data.history.map(item => ({
          role: item.role,
          content: item.content
        }))
        scrollToBottom()
      }
    } catch (e) {
      console.log("加载历史失败", e)
    }
  }

  // 使用 onMounted() ,页面加载完成，自动执行一些初始化操作
  onMounted(()=>{
    getCat()
    loadHistory() // 新增：加载聊天记录
  })
</script>

<template>
  <div class="app-container">
    
    <div class="sidebar">
      <div class="brand">✨ 猫娘终端</div>

      <div class="panel-card">
        <div class="user-row">
          <el-avatar :size="50" :icon="User" class="user-avatar" />
          <div class="user-info">
            <el-input 
              v-model="name" 
              size="small" 
              placeholder="请输入你的名字喵~"
              :class="{ 'empty-input': !name.trim() }"
            />
            <span class="status" :class="{ 'empty-name': !name.trim() }">
              {{ name.trim() ? '🟢 在线' : '⚪ 离线' }}
            </span>
          </div>
      </div>
    </div>

      <div class="panel-card">
        <div class="panel-title"><el-icon><Trophy /></el-icon> 摸鱼能量值</div>
        <div class="counter-big">{{ count }}</div>
        <div class="btn-row">
          <el-button type="success" @click="add" style="flex:1" round>注入能量</el-button>
          <el-button type="danger" :icon="Delete"  @click="reset" round>能量清零</el-button>
        </div>
      </div>

      <div class="panel-card cat-panel">
        <div class="panel-title">📸 猫猫写真</div>
        <div class="cat-box">
          <el-image :src="catImage" fit="cover" class="cat-img">
            <template #error>
              <div class="img-error">😿 加载失败</div>
            </template>
          </el-image>
        </div>
        <el-button type="warning" plain style="width: 100%; margin-top: 10px;" @click="getCat" :loading="isLoading" :icon="Refresh" round>
          换一张喵
        </el-button>
      </div>

      <div class="log-panel">
        <div v-for="(log, i) in logs.slice(0, 6)" :key="i" class="log-line">
          {{ log }}
        </div>
      </div>
    </div>

    <div class="chat-area">
      <div class="chat-header">
        <span>💖 与猫娘的私密对话</span>
      </div>

      <div class="messages" ref="chatBoxRef">
        <div v-for="(msg, i) in chatHistory" :key="i" 
             class="msg-row" :class="msg.role === 'user' ? 'msg-right' : 'msg-left'">
          
          <div class="bubble" :class="msg.role === 'user' ? 'bubble-user' : 'bubble-assistant'">
              <!-- 如果是AI消息且正在思考且内容为空，显示思考提示 -->
              <template v-if="msg.role === 'assistant' && isThinking && msg.content === ''&& chatHistory[chatHistory.length - 1] === msg">
                🐱 正在思考喵...
              </template>
              <!-- 否则显示实际内容 -->
              <template v-else>
                <div v-html="renderMarkdown(msg.content)"></div>
              </template>
          </div>
        </div>
      </div>

      <div class="input-box">
        <el-input 
          v-model="chatInput" 
          placeholder="和猫娘说点什么吧..." 
          size="large" 
          @keyup.enter="sendMessage"
          :disabled="isChatting"
        >
          <template #append>
            <el-button @click="sendMessage" :loading="isChatting" type="primary">发送喵</el-button>
          </template>
        </el-input>
      </div>
    </div>

  </div>
</template>

<style scoped>
/* === 全局布局 === */
.app-container {
  display: flex;
  width: 100vw;
  height: 100vh;
  background-color: #fffafc; /* 极淡的粉色背景 */
  overflow: hidden;
  font-family: 'Comic Sans MS', '幼圆', sans-serif; /* 尝试用一点可爱的字体，如果没有就回退 */
}

/* === 左侧样式 === */
.sidebar {
  width: 300px;
  background-color: #fff0f5; /* 淡粉色侧边栏 */
  border-right: 2px solid #ffd1dc;
  display: flex;
  flex-direction: column;
  padding: 20px;
  gap: 20px;
  overflow-y: auto;
}

.brand {
  font-size: 18px;
  font-weight: bold;
  color: #ff69b4; /* 热粉色 */
  text-align: center;
  padding-bottom: 10px;
}

.panel-card {
  background: #fff;
  padding: 15px;
  border-radius: 16px;
  border: 2px solid #ffe4e1; /* 浅玫瑰色边框 */
  box-shadow: 0 4px 12px rgba(255, 182, 193, 0.2); /* 粉色阴影 */
}

.panel-title {
  font-size: 14px;
  color: #ff69b4;
  margin-bottom: 10px;
  display: flex;
  align-items: center;
  gap: 5px;
}

/* 用户信息 */
.user-row { display: flex; align-items: center; gap: 10px; }
.status.empty-name {
  color: #c0c4cc;
}

.user-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.status { font-size: 12px; color: #67c23a; margin-left: 5px; }
.user-avatar { background: #e6dadd; }

.counter-big {
  font-size: 48px;
  font-weight: 800;
  color: #ff69b4;
  text-align: center;
  margin: 10px 0;
  text-shadow: 2px 2px 0px #fff, 4px 4px 0px #ffe4e1;
}
.btn-row { display: flex; gap: 10px; }

.cat-box {
  width: 100%;
  height: 180px;
  border-radius: 12px;
  overflow: hidden;
  border: 2px solid #ffe4e1;
}
.cat-img { width: 100%; height: 100%; }
.img-error { height: 100%; display: flex; align-items: center; justify-content: center; color: #ff99cc; }

.log-panel {
  font-size: 12px;
  color: #999;
  padding: 0 5px;
}
.log-line {
  padding: 4px 0;
  border-bottom: 1px dashed #ffd1dc;
}

/* === 右侧聊天样式 === */
.chat-area {
  flex: 1;
  display: flex;
  flex-direction: column;
  background-color: #fffafc;
}

.chat-header {
  height: 60px;
  border-bottom: 2px solid #ffd1dc;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 18px;
  font-weight: bold;
  color: #ff69b4;
  background: #fff0f5;
}

.messages {
  flex: 1;
  padding: 30px;
  overflow-y: auto;
  background-image: radial-gradient(#ffe4e1 10%, transparent 10%),
                    radial-gradient(#ffe4e1 10%, transparent 10%);
  background-size: 30px 30px;
  background-position: 0 0, 15px 15px; /* 可爱的波点背景 */
}

.msg-row {
  display: flex;
  margin-bottom: 25px;
  align-items: flex-start;
}
.msg-left { justify-content: flex-start; }
.msg-right { justify-content: flex-end; }

/* === 核心：二次元气泡样式 === */
.bubble {
  max-width: 75%;
  padding: 15px 20px;
  border-radius: 20px;
  font-size: 15px;
  line-height: 1.6;
  position: relative;
  box-shadow: 4px 4px 0px rgba(0,0,0,0.05); /* 漫画感硬阴影 */
  border: 3px solid; /* 粗边框 */
}

/* 猫娘气泡 (粉色) */
.bubble-assistant {
  background: #fff;
  color: #333;
  border-color: #ffb6c1; /* 浅粉色边框 */
  margin-left: 15px; /* 给尖角留位置 */
  border-top-left-radius: 4px;
}
/* 猫娘气泡尖角 (用 CSS 画三角形) */
.bubble-assistant::before {
  content: '';
  position: absolute;
  left: -18px; top: 15px;
  border-width: 10px 18px 10px 0;
  border-style: solid;
  border-color: transparent #ffb6c1 transparent transparent;
  z-index: 1;
}
.bubble-assistant::after {
  content: '';
  position: absolute;
  left: -13px; top: 15px;
  border-width: 10px 18px 10px 0;
  border-style: solid;
  border-color: transparent #fff transparent transparent;
  z-index: 2;
}

/* 用户气泡 (蓝色) */
.bubble-user {
  background: #fff;
  color: #333;
  border-color: #87ceeb; /* 天蓝色边框 */
  margin-right: 15px; /* 给尖角留位置 */
  border-top-right-radius: 4px;
}
/* 用户气泡尖角 */
.bubble-user::before {
  content: '';
  position: absolute;
  right: -18px; top: 15px;
  border-width: 10px 0 10px 18px;
  border-style: solid;
  border-color: transparent transparent transparent #87ceeb;
  z-index: 1;
}
.bubble-user::after {
  content: '';
  position: absolute;
  right: -13px; top: 15px;
  border-width: 10px 0 10px 18px;
  border-style: solid;
  border-color: transparent transparent transparent #fff;
  z-index: 2;
}

/* ✅ 新增：Markdown 样式修正 */
/* :deep() 是为了穿透 v-html 生成的内容 */
.bubble :deep(p) {
  margin: 0; /* 去掉段落默认的间距 */
  padding: 0;
  display: inline; /* 让文字紧凑 */
}

.bubble :deep(strong) {
  font-weight: bold; /* 确保加粗显示 */
  color: #ff1493; /* 给加粗文字一点特别的颜色(可选)，比如深粉色 */
}

.bubble :deep(a) {
  color: #409eff; /* 链接颜色 */
  text-decoration: underline;
}

.input-box {
  padding: 20px;
  border-top: 2px solid #ffd1dc;
  background: #fff0f5;
}
/* 修改输入框按钮颜色 */
:deep(.el-input-group__append button.el-button) {
  background-color: #ff69b4 !important;
  border-color: #ff69b4 !important;
  color: white !important;
  border-top-left-radius: 0;
  border-bottom-left-radius: 0;
}

/* === 手机端适配 === */
@media (max-width: 768px) {
  .app-container { flex-direction: column; }
  .sidebar {
    width: 100%; height: auto;
    border-right: none; border-bottom: 2px solid #ffd1dc;
    padding: 15px;
  }
  .log-panel { display: none; }
  .chat-area { height: 60vh; }
  .messages { padding: 15px; }
}
</style>