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
const userObj = ref();

onMounted(async () => {
  if (getCookie("_id")) {
    const id = getCookie("_id");
    const response = await axios.get(
      `https://to-do-list-server-amber.vercel.app/api/users?key=${API_KEY}&_id=${id}`
    );
    userObj.value = response.data;
    console.log(userObj.value);
    isRegistered.value = true;
    isLoginForm.value = false;

    const tasks = await axios.get(
      `https://to-do-list-server-amber.vercel.app/api/tasks?key=${API_KEY}&userId=${id}`
    );
    todoes.value = tasks.data.data;
    console.log(todoes.value);
    // console.log(tasks);
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
        `https://to-do-list-server-amber.vercel.app/api/users?login=${login.value}&password=${password.value}&key=${API_KEY}`
      )
      .then(async (res) => {
        const userData = res.data;
        if (userData.status && userData.status !== 200) {
          formError.value = userData.message;
        } else {
          userObj.value = userData.data;
          isLoginForm.value = false;
          isRegistered.value = true;
          if (!getCookie("_id")) setCookie("_id", userData.data._id, 3);
          const tasks = await axios.get(
            `https://to-do-list-server-amber.vercel.app/api/tasks?key=${API_KEY}&userId=${userData.data._id}`
          );
          todoes.value = tasks.data.data;
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
        `https://to-do-list-server-amber.vercel.app/api/users?key=${API_KEY}`,
        data
      );
      const res = await req;
      const object = await res.data;
      console.log(object);
      if (res.status == 200) {
        if (!getCookie("_id")) setCookie("_id", object.data._id, 3);
        userObj.value = object;
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

  console.log(title.value.length);

  if (todoes.value.length < 7) {
    if (title.value.length < 30) {
      axios
        .post(
          `https://to-do-list-server-amber.vercel.app/api/tasks?key=${API_KEY}`,
          {
            title: title.value,
            striked: false,
            userId: userObj.value._id,
          }
        )
        .then((task) => {
          console.log(task);
          todoes.value.push(task.data.data);
          console.log(todoes.value);
        })
        .catch((err) => {
          console.log(err);
        });
    } else {
      alert("Слишком много символов");
    }
  } else {
    alert("Уже слишком много задач");
  }
}
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
  gap: 100px;
  button {
    height: fit-content;
  }
}

.todo {
  display: flex;
  flex-direction: column;
  gap: 40px;

  input {
    width: 100%;
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

@media (max-width: 767px) {
  .todo__header {
    flex-direction: column;
    gap: 20px;
  }

  .todo__container {
    padding: 30px;
  }
}
</style>
