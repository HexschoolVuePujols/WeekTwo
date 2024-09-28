<template>
  <div class="container">
    <div>
      <h2>註冊</h2>
      <input type="text" name="registerEmail" id="registerEmail" placeholder="Email" v-model="registerData.email">
      <input type="password" name="registerPassword" id="registerPassword" placeholder="Password"
        v-model="registerData.password">
      <input type="text" name="registerNickname" id="registerNickname" placeholder="Nickname"
        v-model="registerData.nickname">
      <button type="button" @click="register">Sign Up</button>
      <br>
      <div v-if="uid">
        uid: {{ uid }}
      </div>
    </div>

    <div>
      <h2>登入</h2>
      <input type="text" name="signinEmail" id="signinEmail" placeholder="Email" v-model="signInData.email">
      <input type="password" name="signinPassword" id="signinPassword" placeholder="Password"
        v-model="signInData.password">
      <button type="button" @click="signIn">Sign In</button>
      <br>
      Token: {{ token }}
    </div>

    <div>
      <h2>驗證</h2>
      <input type="text" name="token" id="token" v-model="verifiedToken">
      <button type="button" @click="verifyToken">Check Out</button>
      <br>
      <div v-if="verifyData.isVerified">
        {{ verifyData.message }}
      </div>
    </div>

    <div>
      <h2>登出</h2>
      <input type="text" name="signoutToken" id="signoutToken" v-model="signoutToken" placeholder="Token">
      <button type="button" @click="signOut">Sign Out</button>
    </div>

    <hr>

    <div>
      <h2>Todo list</h2>
      <input type="text" name="newTodo" id="newTodo" v-model="newTodo">
      <button type="button" @click="addTodo">Add Todo</button>
      <p>
      <ul>
        <li v-for="todo in todos" :key="todo.id">
          {{ todo.content }} {{ todo.status ? '完成' : '未完成' }} | <input type="text" name="" id="" placeholder="更新值"
            v-model="todo.editedContent">
          <button type="button" @click="deleteTodo(todo.id)">Delete</button>
          <button type="button" @click="updateTodo(todo.id, todo.editedContent)">Update</button>
          <button type="button" @click="toggleTodo(todo.id)">Toggle Ststus</button>
        </li>
      </ul>
      </p>
    </div>
  </div>

</template>

<script setup>
// import HelloWorld from './components/HelloWorld.vue'
// import TheWelcome from './components/TheWelcome.vue'
import { ref } from 'vue'
import axios from 'axios'

const baseApiUrl = 'https://todolist-api.hexschool.io'

const registerData = ref({
  email: '',
  password: '',
  nickname: ''
});
const uid = ref('')
const register = async () => {
  try {
    const response = await axios.post(`${baseApiUrl}/users/sign_up`, {
      email: registerData.value.email,
      password: registerData.value.password,
      nickname: registerData.value.nickname
    });
    console.log("register response", response)

    if (response.data.status) {
      uid.value = response.data.uid;
    }

  } catch (error) {
    console.log("register error", error);
  }
}

const signInData = ref({
  email: '',
  password: ''
});
const token = ref('');
const tokenExp = ref('');
const nickname = ref('');
const signIn = async () => {
  try {

    const response = await axios.post(`${baseApiUrl}/users/sign_in`, {
      email: signInData.value.email,
      password: signInData.value.password
    });

    console.log("SignIn response", response);

    if (response.data.status) {
      token.value = response.data.token;
      tokenExp.value = response.data.exp;
      nickname.value = response.data.nickname;

      getTodos();
    }

  } catch (error) {
    console.log("signIn error", error);
  }
};

const verifiedToken = ref('');
const verifyData = ref({
  isVerified: false,
  result: '',
  message: ''
});
const verifyToken = async () => {

  try {
    const response = await axios.get(`${baseApiUrl}/users/checkout`, {
      headers: {
        authorization: verifiedToken.value
      }
    })

    console.log("verifyToken response,", response);

    if (response.data.status) {
      verifyData.value.isVerified = true;
      uid.value = response.data.uid
      verifyData.value.message = `驗證成功 UID: ${uid.value}`;
    } else {
      verifyData.value.isVerified = true;

      verifyData.value.message = "驗證失敗";

    }

  } catch (error) {
    console.log("verifyToken error,", error);
    verifyData.value.isVerified = true;
    verifyData.value.message = `驗證失敗: ${error.message}`;
  }


}

const signoutToken = ref('');
const signOut = async () => {

  try {
    const response = await axios.post(`${baseApiUrl}/users/sign_out`, {}, {
      headers: {
        authorization: signoutToken.value
      }
    });

    console.log("signOut response,", response);
    if (response.data.status) {
      alert(`登出成功`);

    } else {
      alert(`登出失敗: ${response.data.message}`);

    }


  } catch (error) {
    console.log("signOut error,", error);
    alert(`登出失敗: ${error.message}`);
  }


};

const newTodo = ref('');
const todos = ref([]);

const getTodos = async () => {

  try {
    const response = await axios.get(`${baseApiUrl}/todos/`, {
      headers: {
        authorization: token.value
      }
    })

    console.log("getTodos response", response)

    if (response.data.status) {
      todos.value = response.data.data;

      for (let i = 0; i < todos.value.length - 1; i++) {
        todos.value.editedContent = todos.value.content
      }

    }

  } catch (error) {
    console.log("getTodos error", error)
  }
};

const addTodo = async () => {
  if (newTodo.value == '' || newTodo.value == null) {
    return;
  }

  try {
    const response = await axios.post(`${baseApiUrl}/todos/`, { content: newTodo.value }, {
      headers: {
        authorization: token.value
      }
    });

    console.log("addTodo response", response);

    if (response.data.status) {
      todos.value.push({ ...response.data.newTodo, editedContent: newTodo.value });

    } else {

    }


    newTodo.value = '';
  } catch (error) {

    console.log("addTodo error", error);

  }



};

const updateTodo = async (id, content) => {
  try {
    const response = await axios.put(`${baseApiUrl}/todos/${id}`,
      {
        content: content
      },
      {
        headers: {
          authorization: token.value
        }
      });

    console.log("updateTodo response", response);

    if (response.data.status) {
      // 這時候再去更新 todos 裡面的 目標 todo 的 content

      const index = todos.value.findIndex(item => item.id === id);
      todos.value[index].content = content;

      todos.value[index].editedContent = ""

    }

  } catch (error) {
    console.log("updateTodo error", error);

  }

};


const deleteTodo = async (id) => {
  try {
    const response = await axios.delete(`${baseApiUrl}/todos/${id}`, {
      headers: {
        authorization: token.value
      }
    });

    console.log("deleteTodo response", response);

    if (response.data.status) {
      const index = todos.value.findIndex(item => item.id === id);
      todos.value.splice(index, 1);
    }

  } catch (error) {
    console.log("deleteTodo error", error);

  }
}

const toggleTodo = async (id) => {
  try {
    const response = await axios.patch(`${baseApiUrl}/todos/${id}/toggle`, {}, {
      headers: {
        authorization: token.value
      }
    });
    if (response.data.status) {
      const index = todos.value.findIndex(item => item.id === id);
      todos.value[index].status = !todos.value[index].status;
    }
  } catch (error) {

  }
}


</script>


<style scoped>
.container {
  margin-left: 20%;
  margin-right: 20%;
}
</style>
