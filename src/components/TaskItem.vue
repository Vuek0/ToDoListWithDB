<script setup>
import axios from "axios";
import { ref, watch } from "vue";
import FormInput from "./FormInput.vue";
import PrimaryButton from "./PrimaryButton.vue";
const task__value = ref(null);
const API_KEY = import.meta.env.VITE_API_KEY;
const props = defineProps({
  taskId: String,
  title: String,
  arr: Array,
  striked: Boolean,
  // msg: String,
});
const editValue = defineModel("editValue");
editValue.value = props.title;
const isEditing = ref();
const isChecked = ref(props.striked);
const title = ref(props.title);
// console.log(props);
function openEditing() {
  editValue.value = props.title;
  isEditing.value ? (isEditing.value = false) : (isEditing.value = true);
}
function checkTask(e) {
  props.arr.forEach((item, index, arr) => {
    if (item._id == props.taskId) {
      console.log(item.checked);
      if (item.striked == false) {
        axios.put(`http://localhost:2000/api/tasks?key=${API_KEY}`, {
          _id: props.taskId,
          title: props.title,
          striked: true,
        });
        isChecked.value = true;
        item.striked = true;
      } else {
        axios.put(`http://localhost:2000/api/tasks?key=${API_KEY}`, {
          _id: props.taskId,
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

function editTask(e) {
  e.preventDefault();
  if (editValue.value !== props.title) {
    if (editValue.value.length < 30) {
      axios
        .put(`http://localhost:2000/api/tasks?key=${API_KEY}`, {
          _id: props.taskId,
          title: editValue.value,
          striked: props.striked,
        })
        .then((data) => {
          title.value = editValue.value;
          isEditing.value = false;
        })
        .catch((err) => {
          console.log(err);
        });
    } else {
      alert("Слишком много символов");
    }
  }
}

function deleteTask() {
  // alert("deleting");
  const id = props.taskId;
  axios
    .delete(`http://localhost:2000/api/tasks?key=${API_KEY}&_id=${id}`)
    .then((data) => {
      console.log(data);
    });
  props.arr.forEach((item, index, arr) => {
    if (item._id == props.taskId) {
      arr.splice(index, 1);
    }
  });
}
</script>

<template>
  <div class="todo__task">
    <div :class="['task__check', `${isChecked}`]" @click="checkTask"></div>
    <p
      class="task__value"
      :class="`${isChecked}`"
      ref="task__value"
      v-if="!isEditing"
    >
      {{ title }}
    </p>
    <form class="editing" v-else-if="isEditing" @submit="editTask">
      <FormInput v-model="editValue" :value="editValue" />
      <PrimaryButton text="Edit" />
    </form>

    <div class="task__buttons">
      <div class="task__edit" @click="openEditing">
        <img
          src="/edit-nonactive.svg"
          alt="edit"
          class="task__edit__icon"
          width="36px"
          height="36px"
          @mouseenter="
            (e) => {
              e.srcElement.src = '/edit-active.svg';
            }
          "
          @mouseleave="
            (e) => {
              e.srcElement.src = '/edit-nonactive.svg';
            }
          "
        />
      </div>
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
  </div>
</template>

<style lang="scss" scoped>
.todo__task {
  display: flex;
  align-items: center;
  gap: 20px;
}

.editing {
  display: flex;
  align-items: center;

  input {
    height: 40px;
    border-radius: 10px;
    border-top-right-radius: 0;
    border-bottom-right-radius: 0;
  }

  button {
    border-radius: 10px;
    border-top-left-radius: 0;
    border-bottom-left-radius: 0;
  }
}
.todo__task:hover .task__value {
  color: var(--primary-color);
}
.task__value.true {
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
}

.task__edit {
  cursor: pointer;
}

.task__buttons {
  display: flex;
  /* align-items: center; */
  gap: 20px;
  margin-left: auto;
}

.task__check.true {
  background: var(--success-color);
  background-image: url(check-mark.svg);
}
</style>
