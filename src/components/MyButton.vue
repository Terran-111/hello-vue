<script setup>
  // 接收外部传进来的文字
  defineProps(['text'])
</script>

<template>
  <button class="fancy-btn">
    {{ text }} 
  </button>
</template>

<style scoped>
  .fancy-btn {
    background: linear-gradient(45deg, #42b883, #35495e);
    border: none;
    color: white;
    padding: 12px 24px;
    text-align: center;
    font-size: 16px;
    margin: 4px 2px;
    cursor: pointer;
    border-radius: 25px; /* 圆角 */
    transition: 0.3s;
    box-shadow: 0 4px 6px rgba(0,0,0,0.1);
  }
  
  .fancy-btn:hover {
    transform: scale(1.05); /* 鼠标放上去变大一点点 */
    box-shadow: 0 6px 8px rgba(0,0,0,0.2);
  }
</style>