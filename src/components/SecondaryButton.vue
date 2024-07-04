<script setup>
defineProps({
  id: String,
  text: String,
  func: Function,
});
</script>

<template>
  <button type="submit" class="primary-btn" :id="id" @click="func">
    {{ text }}
  </button>
</template>

<style scoped>
.primary-btn {
  width: fit-content;
  background: var(--danger-color);
  border: none;
  border-radius: 5px;
  padding: 10px;
  color: #fff;
  font-size: 18px;
  cursor: pointer;
  transition: 0.15s;
}
</style>
