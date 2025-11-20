<script setup>
  import { ref } from 'vue' // 引入 Vue 的核心魔法棒

  // 定义一个“会动的变量” (响应式变量)
  // ref(0) 意思是初始值是 0
  const count = ref(0)

  // 定义一个函数：点击后执行什么
  function add() {
    count.value = count.value + 1
    console.log("现在的数字是：", count.value)
  }
  function reset(){
    count.value=0
  }
</script>

<template>
  <div class="container">
    <h1>Vue 计数器</h1>
    
    <p class="number">{{ count }}</p>

    <button @click="add">点我 +1</button>
    <button @click="reset">重置</button>
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