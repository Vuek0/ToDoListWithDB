<script setup>
defineProps({
  id: String,
  placeholder: String,
});

defineEmits(["update:modelvalue"]);
</script>

<template>
  <input
    type="text"
    class="form__input"
    :placeholder="placeholder"
    :id="id"
    @input="$emit('update:modelValue', $event.target.value)"
  />
</template>

<style scoped>
.form__input {
  width: 600px;
  padding: 18px 20px;
  outline: none;
  font-size: 18px;
  border: 1px solid lightgray;
  border-radius: 5px;
}
</style>
