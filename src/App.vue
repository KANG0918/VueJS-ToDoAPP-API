<script setup>
import { ref,onMounted,computed } from 'vue';

const token = window.localStorage.getItem('token') || '';
const memberInfo = ref({
  nickname: '',
  email: '',
  password: '',
});

const userLogin = () => {
  fetch('https://todoo.5xcamp.us/users/sign_in', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': token,
    },
    // 傳入正確的字串
    body: JSON.stringify({
      user: {
        email: memberInfo.value.email,
        password: memberInfo.value.password,
      }
    }),
  })
    .then((res) => {
      const headers = res.headers;
      const authorization = headers.get('Authorization');
      window.localStorage.setItem('token', authorization);
      return res.json();
    })
    .then((data) => {
      memberInfo.value.nickname = data?.nickname ||'';
      if(memberInfo.value.nickname !== ''){
        getList()

      }
    });
}


const todos=ref([]) 
const getList=async()=>{
  const res=await fetch('https://todoo.5xcamp.us/todos',{
    method: 'GET',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': token,
    },
  })
  const data=await res.json()
  todos.value=data.todos

}

const newTodo=ref('')
const addItem = async () => {
  await fetch('https://todoo.5xcamp.us/todos', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': token,
    },
    body: JSON.stringify({
      "todo": {
        "content": newTodo.value,
      }
    }),
  });
  newTodo.value=''
  getList();
}


const deleteItem = async (id) => {
  await fetch(`https://todoo.5xcamp.us/todos/${id}`, {
    method: 'DELETE',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': token,
    }
  });

  getList();
}

const toggleItem = async (id) => {
  await fetch(`https://todoo.5xcamp.us/todos/${id}/toggle`, {
    method: 'PATCH',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': token,
    }
  });

  getList();
}

const searchName=ref('')
const filteredTodos=computed(()=>{
  return todos.value.filter((item)=>{
    return item.content.toLowerCase().includes(searchName.value.toLowerCase())
  })
})

onMounted(()=>{
    userLogin()
  }
)

</script>

<template>
  <div class="bg-gray-100 pb-12">
    <!-- Header -->
    <header class="bg-blue-600 text-white p-4">
      <div class="container flex mx-auto justify-between items-center">
        <h1 class="font-bold text-xl">Todo List</h1>
        <div>
          <button class="rounded bg-blue-800 py-2 px-4" v-if="token ===''">Login</button>
          <button class="rounded bg-blue-800 py-2 px-4" v-if="token ===''">Register</button>
          <span class="py-2 px-4" v-if="token !==''">{{memberInfo.nickname}}</span>
          <button class="rounded bg-blue-800 py-2 px-4" v-if="token !==''">Logout</button>
        </div>
        
      </div>
    </header>

    <!-- Main Content -->
    <main class="container mx-auto mt-8">
      <!-- Add Todo Item -->
      <div class="bg-white rounded shadow p-6">
        <h2 class="font-semibold text-lg mb-4">Add New Todo</h2>
        
          <input v-model="newTodo" type="text" id="todoInput" class="border w-full p-2" placeholder="Enter new todo item">
          <button @click.prevent="addItem()" type="submit" class="rounded bg-blue-600 mt-2 text-white py-2 px-4">Add</button>
        
      </div>

      <!-- Todo List -->
      <div class="bg-white rounded shadow mt-6 p-6">
        <h2 class="font-semibold text-lg mb-4">Todo List</h2>
        <input class="border border-gray px-2 py-1" v-model="searchName">
        <ul class="list-disc pl-5">          
          <li v-for="item in filteredTodos" class="flex mb-2 justify-between items-center">
            <span :class="{'line-through text-gray-400': item.completed_at !== null}">{{item.content}}</span>
            <button class="rounded bg-green-600 text-white py-1 px-2" @click="toggleItem(item.id)">Finish</button>
            <button class="rounded bg-red-600 text-white py-1 px-2" @click="deleteItem(item.id)">Delete</button>
          </li>
          
        </ul>
      </div>
    </main>

    <!-- Modals -->

    <!-- Login Modal -->
    <div
      v-if="memberInfo.nickname === ''"
      class="flex bg-gray-600 bg-opacity-50 inset-0 fixed items-center justify-center">
      <div class="bg-white rounded shadow p-6">
        <h2 class="font-semibold text-lg mb-4">Login</h2>
        <div>
          <input v-model="memberInfo.email" type="email" class="border mb-4 w-full p-2" placeholder="Email">
          <input v-model="memberInfo.password" type="password" class="border mb-4 w-full p-2" placeholder="Password">
          
          <button @click="userLogin"
            class="rounded bg-blue-600 text-white py-2 px-4">Login</button>
        </div>
      </div>
    </div>
  </div>
</template>
