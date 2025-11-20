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
  <div class="container">
    <h1>Hello,{{ name }}</h1>
    <input v-model="name" type="text" placeholder="请输入你的名字" style="padding: 10px;font-size: 18px;"></input>
    
    <p class="number">{{ count }}</p>

    <h2 v-if="count>=10" style="color:red">🎉 恭喜你达成目标！</h2>

    <div class="buttons">
      <MyButton @click="add" text="点我 +1" />
      <MyButton @click="reset" text="重置" v-if="count > 0"/>
    </div>


    <hr style="margin: 30px 0;">
      <div style="text-align: center;">
        <h3>🐱 每日吸猫</h3>
        <img :src="catImage" style="width: 300px; height: 300px;"/>
        <br><br>
        <MyButton text="换一只猫" @click="getCat"/>
      </div>
    <hr style="margin: 30px 0;">
    
    <!-- <button @click="add">点我 +1</button>
    <button @click="reset" v-if="count>0" style="background-color: #e74c3c;">重置</button>
   -->
    <div style="margin-top: 30px;text-align: left;">
      <h3>📜 操作日志</h3>
      <ul>
        <li v-for="(item,index) in logs">
          第{{ index+1 }}次操作：{{ item}}
        </li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
  .container {
    text-align: center;
    margin-top: 60px;
    font-family: sans-serif;
  }
  .number {
    font-size: 80px;
    color: #42b883;
    font-weight: bold;
    margin: 20px 0;
  }
  button {
    font-size: 20px;
    padding: 10px 30px;
    cursor: pointer;
    background-color: #333;
    color: white;
    border: none;
    border-radius: 5px;
  }
  button:hover {
    background-color: #555;
  }
</style>