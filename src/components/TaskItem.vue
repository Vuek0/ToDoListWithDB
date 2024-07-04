<script setup>
import axios from "axios";
import { ref, watch } from "vue";
const task__value = ref(null);
const API_KEY = import.meta.env.VITE_API_KEY;
const props = defineProps({
  taskId: Number,
  title: String,
  arr: Array,
  striked: Boolean,
  // msg: String,
});
const isChecked = ref(striked.value);
const emit = defineEmits(["deleteHandler"]);
function checkTask(e) {
  props.arr.forEach((item, index, arr) => {
    if (item.id == props.taskId) {
      console.log(item.checked);
      if (item.striked == false) {
        axios.put(`http://localhost:2000/api/posts?key=${API_KEY}`, {
          _id: taskId,
          title: props.title,
          striked: true,
        });
        isChecked.value = true;
        item.striked = true;
      } else {
        axios.put(`http://localhost:2000/api/posts?key=${API_KEY}`, {
          _id: taskId,
          title: props.title,
          striked: false,
        });
        isChecked.value = false;
        item.striked = false;
      }
      props.arr = arr;
    }
  });
}

function addcheckedToArr() {}

function deleteTask() {
  axios.delete(`http://localhost:2000/api/tasks?key=${API_KEY}`, {
    _id: props.taskId,
  });
  props.arr.forEach((item, index, arr) => {
    if (item.id == props.taskId) {
      arr.splice(index, 1);
      localStorage.setItem("tasks", JSON.stringify(arr));
    }
  });
  emit("deleteHandler");
}
</script>

<template>
  <div class="todo__task">
    <div :class="['task__check', `${isChecked}`]" @click="checkTask"></div>
    <p class="task__value" :class="`${isChecked}`" ref="task__value">
      {{ title }}
    </p>
    <div class="task__delete" @click="deleteTask">
      <img
        src="/delete-nonactive.svg"
        alt="delete"
        class="task__delete__icon"
        width="36px"
        height="36px"
        @mouseenter="
          (e) => {
            e.srcElement.src = '/delete-active.svg';
          }
        "
        @mouseleave="
          (e) => {
            e.srcElement.src = '/delete-nonactive.svg';
          }
        "
      />
    </div>
  </div>
</template>

<style scoped>
.todo__task {
  display: flex;
  align-items: center;
  gap: 20px;
}
.todo__task:hover .task__value {
  color: var(--primary-color);
}
.task__value.checked {
  text-decoration: line-through;
  color: gray;
}

.task__value:active {
  transition: 0.6s;
}
.task__check {
  width: 30px;
  height: 30px;
  border: 1px solid lightgray;
  border-radius: 100%;
  background-repeat: no-repeat !important;
  background-position: 7px 9px !important;
  cursor: pointer;
  transition: 0.6s;
}

.task__delete {
  cursor: pointer;
  display: flex;
  margin-left: auto;
}

.task__check.checked {
  background: var(--success-color);
  background-image: url(check-mark.svg);
}
</style>
