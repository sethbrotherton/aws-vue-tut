<template>
  <div>
    <div>
      <h1>Welcome to the amplify framework</h1>
    </div>
    <amplify-authenticator>
      <div>
        <h1>Hey, {{user.username}}!</h1>
        <amplify-sign-out></amplify-sign-out>
      </div>
      <div>
        <form v-on:submit.prevent>
<!--          <button @click='getTodos'>GET 'todos'</button>-->
<!--          <button @click='getTodo'>GET 'todo'</button>-->
<!--          <button @click='addTodo'>POST 'todos'</button>-->
<!--          <button @click='updateTodo'>PUT 'todos'</button>-->
<!--          <button @click='deleteTodo'>DELETE 'todos'</button>-->
          <input type="text" v-model="nextTodo"/>
          <p>{{nextTodo}}</p>
          <button type="button" @click="addTodo">Add new todo</button>
        </form>
      </div>
      <h2>To do list</h2>
      <div>
        <div v-for="todo of todos" :key="todo.id">
          <p>{{todo.text}}</p>
            <button v-if="!todo.complete" @click="markAsComplete(todo)">Mark as complete</button>
            <p v-if="todo.complete"><strong>Completed!!!</strong></p>
          <button @click="deleteTodo(todo.id)">Delete</button>
        </div>
      </div>
    </amplify-authenticator>
  </div>
</template>

<script>
import { AuthState, onAuthUIStateChange } from '@aws-amplify/ui-components';
import { API } from 'aws-amplify';

export default {
  name: 'App',
  data() {
    return {
      user: {},
      todos: [],
        nextTodo: ''
    }
  },
  created() {
    onAuthUIStateChange((state, user) => {
      if (state === AuthState.SignedIn) {
        this.user = user;
        console.log(user);
        this.getTodos();
      }
    })
  },
  methods: {
    getTodos: function() {
      console.log("getTodos");
      API.get('todosApi', '/todos', {}).then(result => {
        console.log(result);
        this.todos = result.body;
        console.log("todos are ", this.todos)
      }).catch(err => {
        console.log('Err', err);
      })
    },
    addTodo: function() {
      console.log("addtodo");
      API.post('todosApi', '/todos', {
        body: {
          text: this.nextTodo
        }
      }).then(result => {
        console.log(result);
        this.todos.push(result.body);
        this.nextTodo = '';
      }).catch(err => {
        console.log('Err', err);
      })
    },
    deleteTodo: function(id) {
      API.del('todosApi', `/todos/${id}`, {}).then(res => {
        console.log('Res', res);
        this.todos = this.todos.filter(todo => todo.id !== id);
      }).catch(err => {
        console.log('Err', err);
      })
    },
      markAsComplete: function(todo) {
        API.put('todosApi', `/todos/${todo.id}`, {
            body: {
                ...todo,
                complete: true
            }
        }).then(res => {
            const index = this.todos.findIndex(item => item.id === todo.id);
            // this.todos = [...this.todos.splice(index, 1, res.body)];
            this.todos[index] = res.body;
            this.todos = [...this.todos];
        }).catch(err => {
            console.log('Err', err);
        })
      }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
