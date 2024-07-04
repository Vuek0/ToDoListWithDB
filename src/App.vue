<script setup>
import { onMounted, ref, watch } from "vue";
import TaskItem from "./components/TaskItem.vue";
import PrimaryButton from "./components/PrimaryButton.vue";
import FormInput from "./components/FormInput.vue";
import { getCookie, setCookie } from "./functions/Cookies";
import axios from "axios";
import SecondaryButton from "./components/SecondaryButton.vue";
const isRegistered = ref(false);
const isLoginForm = ref(true);
const login = defineModel("login");
const password = defineModel("password");
const title = ref(null);
const todoes = ref([]);
const formError = ref();
const API_KEY = import.meta.env.VITE_API_KEY;
let userObj;

onMounted(async () => {
  if (getCookie("_id")) {
    const id = getCookie("_id");
    const response = await axios.get(
      `http://localhost:2000/api/users?key=${API_KEY}&_id=${id}`
    );
    const userObj = response.data;
    isRegistered.value = true;
    isLoginForm.value = false;

    const tasks = await axios.get(
      `http://localhost:2000/api/tasks?key=${API_KEY}&userId=${id}`
    );
    todoes.value = tasks;
    console.log(tasks);
  }
});

function exitAccount() {
  setCookie("_id", "", 0);
  window.location.reload();
}

async function loginHandler(e) {
  if (e) {
    e.preventDefault();
  }
  formError.value = "";
  if (login.value && password.value) {
    const response = await axios
      .get(
        `http://localhost:2000/api/users?login=${login.value}&password=${password.value}&key=${API_KEY}`
      )
      .then((res) => {
        const userData = res.data;
        if (userData.status && userData.status !== 200) {
          formError.value = userData.message;
        } else {
          userObj = userData.data;
          isLoginForm.value = false;
          isRegistered.value = true;
          if (!getCookie("_id")) setCookie("_id", userData.data._id, 3);
        }
      });
  } else {
    formError.value = "Поля не могут быть пустыми";
    console.log(formError);
  }
}

async function registrationHandler(e) {
  if (e) {
    e.preventDefault();
  }
  if (login.value && password.value) {
    const data = {
      login: login.value,
      password: password.value,
    };
    try {
      const req = await axios.post(
        `http://to-do-list-server-lake.vercel.app/api/users?key=${API_KEY}`,
        data
      );
      const res = await req;
      const object = await res.data.data;
      console.log(object);
      if (res.status == 200) {
        if (!getCookie("_id")) setCookie("_id", object.data._id, 3);
        userObj = object;
        isRegistered.value = true;
      }
    } catch (err) {
      console.log(err);
    }
  } else {
    formError.value = "Поля не могут быть пустыми";
    console.log(formError);
  }
}

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
  axios
    .post(`http://localhost:2000/api/tasks?key=${API_KEY}`, {
      title: title.value,
      striked: false,
      userId: userObj._id,
    })
    .then((task) => {
      todoes.value.push(task);
    })
    .catch((err) => {
      console.log(err);
    });

  // const arr = [];
  // todoes.value.forEach((item) =>
  //   arr.push({ task: item.task, id: item.id, striked: item.striked })
  // );
  // localStorage.setItem("tasks", JSON.stringify(arr));
  // const get = localStorage.getItem("tasks");
  // console.log(JSON.parse(get));
}

// onMounted(async () => {
//   const tasks = await axios.get("http://localhost:2000/api/tasks");
//   console.log(tasks);
// });

// if (localStorage.getItem("tasks")) {
//   todoes.value = JSON.parse(localStorage.getItem("tasks"));
// }
</script>

<template>
  <div class="wrapper">
    <section class="registration" v-if="!isRegistered && !isLoginForm">
      <form @submit="registrationHandler">
        <h2>Регистрация</h2>
        <div class="formError" v-if="formError">
          <p>{{ formError }}</p>
        </div>
        <FormInput
          type="text"
          placeholder="Логин"
          v-model="login"
          :value="login"
        />
        <FormInput
          type="password"
          placeholder="Пароль"
          v-model="password"
          :value="password"
        />
        <PrimaryButton :text="'Зарегестрироваться'" />
        <p>
          Уже есть аккаунт?
          <a href="#" class="form__link" @click="isLoginForm = true">войти</a>
        </p>
      </form>
    </section>
    <section class="login" v-else-if="isLoginForm">
      <form @submit="loginHandler">
        <h2>Вход</h2>
        <div class="formError" v-if="formError">
          <p>{{ formError }}</p>
        </div>
        <FormInput
          type="text"
          placeholder="Логин"
          v-model="login"
          :value="login"
        />
        <FormInput
          type="password"
          placeholder="Пароль"
          v-model="password"
          :value="password"
        />
        <PrimaryButton :text="'Войти'" />
        <p>
          Еще нет аккаунта?
          <a href="#" class="form__link" @click="isLoginForm = false"
            >Зарегестрироваться</a
          >
        </p>
      </form>
    </section>
    <div class="todo__container" v-if="isRegistered">
      <div class="todo">
        <div class="todo__header">
          <h1>Daily To Do List</h1>
          <SecondaryButton :func="exitAccount" text="Выйти из аккаунта" />
        </div>
        <form id="todo__form">
          <FormInput
            id="todo__input"
            v-model="title"
            placeholder="Add new list item"
          />
          <PrimaryButton text="Add" id="todo__submit" :func="addTask" />
        </form>

        <TaskItem
          :title="item.title"
          :task-id="item._id"
          :striked="item.striked"
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

<style lang="scss" scoped>
h1 {
  font-size: 48px;
  color: #11175e;
}

.wrapper {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100vh;
  max-width: 100vw;
}

.formError {
  background: red;
  color: aliceblue;
  padding: 5px 15px;
  width: 100%;
}

.registration,
.login {
  display: flex;
  flex-direction: column;
  gap: 30px;
  align-items: center;

  h2 {
    font-size: 60px;
  }
  form {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 10px;
    max-width: 250px;
    text-align: center;

    a {
      color: cornflowerblue;

      &:hover {
        color: dodgerblue;
      }
    }
  }
}

.todo__container {
  width: fit-content;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: #fff;
  padding: 100px;
}

.todo__header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  button {
    height: fit-content;
  }
}

.todo {
  display: flex;
  flex-direction: column;
  gap: 40px;

  input {
    width: 600px;
  }
}

#todo__form {
  display: flex;
  align-items: center;
  position: relative;

  button {
    position: absolute;
    left: auto;
    right: 10px;
  }
}

#todo__input::placeholder {
  font-size: 18px;
  color: lightgray;
}

#todo__submit:focus {
  outline: 4px solid lightblue;
}
</style>
