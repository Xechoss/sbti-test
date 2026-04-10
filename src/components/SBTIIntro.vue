<script setup lang="ts">
defineEmits(['start']);
</script>

<template>
  <section id="intro">
    <div class="hero card hero-minimal">
      <h1>MBTI已经过时，SBTI来了。</h1>
      <div class="hero-actions hero-actions-single">
        <button @click="$emit('start')" class="btn-primary">开始测试</button>
      </div>
    </div>
  </section>
</template>
