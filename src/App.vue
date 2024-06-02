<script setup>
import { ref, watch } from "vue";
import TaskItem from "./components/TaskItem.vue";
import PrimaryButton from "./components/PrimaryButton.vue";
import FormInput from "./components/FormInput.vue";
const title = ref(null);
const todoes = ref([]);


function addTask(e) {
  e.preventDefault();
  // if(!title.value){
  //   alert("Задача не может быть пуста");
  //   return;
  // }

  // if(title.value.length > 30){
  //   alert("Задача по длине должна быть не больше 30 символов")
  //   return;
  // }
  const newTitle = title.value;
  if(todoes.value.length === 0 && title.value){
    todoes.value.push({ task: newTitle, id: 1, checked: "no-checked"});
  }
  
  else if (title.value && todoes.value.length < 7) {
    // console.log(todoes.value[todoes.value.length - 1].id);   

    if(todoes.value[todoes.value.length - 1].id != 7){
      todoes.value.push({ task: newTitle, id: todoes.value.length + 1, checked: "no-checked"});
    } 

    else if(todoes.value[todoes.value.length - 1].id === 7){
      let temp;
      let whichToAdd;
      
      todoes.value.forEach(item => {
        console.log(item.id - temp)
        if(item.id!=1){
          if((item.id - temp) != 1){
            whichToAdd = item.id - 1;
            temp = item.id;
          }
        }
        temp = item.id;
  
      })
      // alert(whichToAdd)
      todoes.value.push({ task: newTitle, id: whichToAdd, checked: "no-checked"});
    }
  }

  const arr = []
  todoes.value.forEach(item=>arr.push({task: item.task, id: item.id, checked: item.checked}));
  localStorage.setItem("tasks", JSON.stringify(arr));
  const get = localStorage.getItem("tasks");
  console.log(JSON.parse(get));
}

if(localStorage.getItem("tasks")){
  todoes.value = JSON.parse(localStorage.getItem("tasks"));
}



</script>

<template>
  <div class="wrapper">
    <div class="todo__container">
      <div class="todo">
        <h1>Daily To Do List</h1>
        <form id="todo__form">
          <FormInput
            id="todo__input"
            v-model="title"
            placeholder="Add new list item"
          />
          <PrimaryButton
            text="Add"
            id="todo__submit"
            :func="addTask"
            style="position: absolute; left: auto; right: 10px"
          />
        </form>

        <TaskItem
          :value="item.task"
          :task-id="item.id"
          :arr="todoes"
          :delete-func="deleteHandler"
          v-for="item in todoes"
          :key="item"
        />
      </div>
    </div>
  </div>
  <!-- <HelloWorld msg="Vite + Vue" /> -->
</template>

<style scoped>
h1 {
  font-size: 48px;
  color: #11175e;
}

.wrapper {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100vh;
}

.todo__container {
  widows: fit-content;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: #fff;
  padding: 100px;
}

.todo {
  display: flex;
  flex-direction: column;
  gap: 40px;
}

#todo__form {
  display: flex;
  align-items: center;
  position: relative;
}

#todo__input::placeholder {
  font-size: 18px;
  color: lightgray;
}

#todo__submit:focus {
  outline: 4px solid lightblue;
}
</style>
