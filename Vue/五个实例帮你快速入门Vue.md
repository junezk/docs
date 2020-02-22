# 五个实例帮你快速入门Vue

## 前言

无论你学习React还是Vue，TodoList是必看的demo，虽然实现的代码很简短，但是它却能涵盖大部分基础知识。

本文分别通过使用Vue的基础知识、组件化知识、工程化知识、路由(vue-router)以及状态管理(vuex)分别实现TodoList。虽然采用了Vue中不同的技术，但是最终的效果基本都是相同的，所以从外观上看也是一个实例。

当然本文只是包含了Vue中很小的一部分知识，主要原因是本人在做前端也是新手，现学现卖的做了一周Vue开发。本文也算了对上周学习的一个总结吧！同时希望能对正在学习或将要学习Vue的同学有所帮助。

千里之行，始于手下！开始实战吧!💪💪💪

## 效果图

![img](五个实例帮你快速入门Vue.assets/170495776d019505)

## 核心功能

1. 添加`待办事项`;
2. `待办事项`列表展示;
3. 根据状态过滤`待办事项`列表;
4. 根据状态统计`待办事项`数量;
5. 修改`待办事项`状态;

## 基础实例

### 关键技术

1. 使用`v-bind`指令以及语法糖`:`;
2. 使用`v-model`指令演示数据绑定;
3. 使用`v-on`指令实现交互以及语法糖`@`;
4. 使用`v-if`、`v-else`指令展示不同元素;
5. 使用`v-for`指令迭代数组;
6. vue构造函数中相关属性的使用;

### 完整代码

```vue
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TodoList</title>
    <style>
        * {
            padding: 0;
            margin: 0;
            box-sizing: border-box;
        }

        #app {
            width: 800px;
            height: 600px;
            margin: 0 auto;
            background-color: rgb(245, 245, 245);
            padding: 24px 0;
        }

        #app .header {
            font-size: 48px;
            text-align: center;
        }

        p,
        div {
            font-size: 16px;
            margin-left: 15px;
            padding: 10px;
        }

        ul,
        li {
            list-style: none;
        }

        label {
            padding: 5px;
            font-size: 13px;
        }

        .txt_input {
            width: 90%;
            height: 30px;
            padding: 10px 10px;
            font-size: 14px;
            border: 1px solid transparent;
        }

        .add_button {
            width: 60px;
            height: 30px;
        }

        li span {
            display: inline-block;
            width: 90%;
        }

        li button {
            font-size: 10px;
            width: 45px;
            height: 25px;
        }
    </style>
</head>

<body>
    <div id="app">
        <header class="header">My Todo List</header>
        <p>当前共有{{todos.length}}个代办事项,已完成{{completed}}个,剩余{{todos.length-completed}}个。</p>

        <p>
            <!--@click='filterList(0) 是 v-on:click='filterList(0)的缩写-->
            <input type="radio" v-model="picked" value="0" @click='filterList(0)'><label>所有</label>
            <input type="radio" v-model="picked" value="1" @click='filterList(1)'><label>已完成</label>
            <input type="radio" v-model="picked" value="2" @click='filterList(2)'><label>未完成</label>
        </p>
        <div class="div">
            <input type="text" class="txt_input" placeholder="请输入代办事项名称" v-model.trim="todo">
            <!--:disabled="isDisabled" 是v-bind:disabled="isDisabled"的缩写-->
            <button :disabled="isDisabled" @click="addTodo" class="add_button">添加</button>
        </div>
        <ul>
            <li v-for="(todo,index) in tempTodos" v-bind:style="setStyle(todo)" :key="index">
                <div>
                    <span>{{index+1}}. {{todo.name}}</span>
                    <button v-if="todo.status" @click="changeStatus(index)">已完成</button>
                    <button v-else="todo.status" @click="changeStatus(index)">未完成</button>
                </div>
            </li>
        </ul>
    </div>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <script>
        var app = new Vue({
            el: '#app', //vue对象挂载带id为app的元素上
            data: {
                todo: '', //待办事项名称
                picked: 0,
                isDisabled: false, //是否可以点击添加按钮
                tempTodos: [],
                // 初始化数据
                todos: [
                    { name: 'JavaScript', status: true },
                    { name: 'Css', status: false },
                    { name: 'Vue', status: true }
                ]
            },
            //计算属性
            computed: {
                //当属性改变时自动计算已完成的待办事项数量
                completed: function () {
                    return this.todos.filter(x => x.status).length;
                }
            },

            methods: {
                //添加代办事项
                addTodo: function () {
                    this.todos.push({ name: this.todo, status: false });
                    this.todo = '';
                },
                //当待办事项未完成时,字体颜色显示为红色
                setStyle: function (obj) {
                    if (!obj.status)
                        return "color:red";
                },
                //修改待办事项状态
                changeStatus: function (index) {
                    this.tempTodos[index].status = !this.tempTodos[index].status;
                    //更新数据
                    this.filterList(parseInt(this.picked));
                },
                //根据条件过滤待办事项
                filterList: function (type) {
                    this.isDisabled = !!type;
                    switch (type) {
                        case 0:
                            this.tempTodos = this.todos;
                            break;
                        case 1:
                            this.tempTodos = this.todos.filter(x => x.status);
                            break;
                        case 2:
                            this.tempTodos = this.todos.filter(x => !x.status);
                            break;
                    }
                }
            },
            //vue生命周期函数，在组件挂载完成后调用
            mounted() {
                this.tempTodos = this.todos;
            }
        });
    </script>
</body>

</html>
```

## 组件化

> 本节将添加`待办事项`功能和`待办事项`展示列表为不同的两个组件。

### 关键技术

1. 如何使用组件；
2. 父组件向子组件传值;
3. 子组件向父组件传值;
4. 无相互关系的组件传值;
5. 如何注册局部组件;
6. 在组件中data属性的变化;

### 完整代码

> html页面以及样式可以直接拷贝**基础实例**章节中的代码,这里只给出变化的部分.

```vue
<body>
    <div id="app">
        <header class="header">My Todo List</header>
        <!--添加待办事项组件-->
        <add-todo @add="add"></add-todo>
        <!--待办事项列表展示组件,向子组件传递属性todos-->
        <todo-list :todos="todos"></todo-list>
    </div>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <script>
        //添加todo组件
        const AddTodo = {
            template: `
                     <div class="div">
                         <input type="text" class="txt_input" placeholder="请输入代办事项名称" v-model.trim="todo">
                         <button :disabled="canAddTodo"  @click="add" class="add_button">添加</button>
                     </div>
                     `,
            data() {
                return {
                    todo: '',
                    canAddTodo: false
                }
            },
            methods: {
                add() {
                    //触发父组件中的add属性绑定的函数
                    this.$emit('add', this.todo);
                    this.todo = '';
                }
            },
            //vue声明周期，在组件初始化之前创建selectChanged监听事件
            created() {
                this.$root.$on('selectChanged', value => {
                    this.canAddTodo = !!value;
                });
            }
        };
        //todo列表展示组件
        const TodoList = {
            template: `
                      <div>
                          <p>
                            <input type="radio" v-model="picked" value="0" @click='filterTodos(0)'><label>所有</label>
                            <input type="radio" v-model="picked" value="1" @click='filterTodos(1)'><label>已完成</label>
                            <input type="radio" v-model="picked" value="2" @click='filterTodos(2)'><label>未完成</label>
                          </p>
                          <p>当前共有{{todoList.length}}个代办事项,已完成{{completed}}个,剩余{{todoList.length-completed}}个。</p>
                          <ul>
                              <li v-for="(todo,index) in tempTodoList" :key="index">
                                  <div>
                                     <span>{{index+1}}. {{todo.name}}</span>
                                     <button v-if="todo.status" @click="changeStatus(index)">已完成</button>
                                     <button v-else="todo.status" @click="changeStatus(index)">未完成</button>
                                  </div>
                              </li>
                          </ul>
                      </div>
                    `,
            props: ['todos'], //接受父组件传递的对象
            // 组件中的data是一个函数
            data: function () {
                return {
                    todoList: this.todos,
                    picked: 0,
                    tempTodoList:[]
                }
            },
            computed: {
                completed() {
                    return this.todoList.filter(x => x.status).length;
                }
            },
            methods: {
                changeStatus(index) {
                    this.tempTodoList[index].status = !this.tempTodoList[index].status;
                    this.filterTodos(parseInt(this.picked));
                },
                filterTodos(type) {
                    //无关系组件之间的通信(触发AddTodo组件中的selectChanged事件)
                    this.$root.$emit('selectChanged', type);
                    switch (type) {
                        case 0:
                            this.tempTodoList = this.todoList;
                            break;
                        case 1:
                            this.tempTodoList = this.todoList.filter(x => x.status);
                            break;
                        case 2:
                            this.tempTodoList = this.todoList.filter(x => !x.status);
                            break;
                    }
                }
            },
            mounted(){
                this.tempTodoList = this.todoList;
            }

        };
        var app = new Vue({
            el: '#app',
            // 注册局部组件
            components: {
                'todo-list': TodoList,
                'add-todo': AddTodo
            },
            data: {
                todos: [
                    { name: 'JavaScript', status: true },
                    { name: 'Css', status: false },
                    { name: 'Vue', status: true }
                ],
                bus: new Vue(),//创建新的Vue实例，用于无关系组件之间的通信
            },
            methods: {
                add(todo) {
                    this.todos.push({ name: todo, status: false });
                }
            }
        });
    </script>
</body>
```

## 工程化

### 目录结构

![img](五个实例帮你快速入门Vue.assets/170498846613a007)

### 关键知识

1. 如何搭建基础工程;
2. webpack基础配置;
3. `.vue`组件文件内容；

### 完整代码

#### 步骤1

```bash
 //初始化
 npm init -y
 
 npm i webpack webpack-cli -g
 
 //注意babel与babel-loader的版本
 npm i babel babel-loader@7 babel-core babel-plugin-transform-runtime babel-preset-es2015 babel-runtime -D
 npm i webpack-dev-server css-loader style-loader -D
 
 npm i vue vue-loader vue-style-loader vue-template-compiler vue-hot-reload-api -S
```

#### 步骤2

webpack配置文件

```js
const path = require('path');
const VueLoaderPlugin = require('vue-loader/lib/plugin');

const config = {
    entry: {
        main: './app.js'
    },
    output: {
        path: path.resolve(__dirname, 'dist'),
        publicPath: '/dist/',
        filename: 'bundle.js'
    },
    module: {
        rules: [
            {
                test: /\.vue$/,
                use: 'vue-loader',
            },
            {
                test: /\.css$/,
                use: ['style-loader', 'css-loader']
            },
            {
                test: /\.js$/,
                use: 'babel-loader',
                exclude: /node_modules/
            }
        ]
    },
    plugins: [
        new VueLoaderPlugin()
    ]
}

module.exports = config;
```

#### 步骤3

编辑index.html文件

```
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <div id="app">
    </div>
    <script type="text/javascript" src="/dist/bundle.js"></script>
</body>

</html>
复制代码
```

#### 步骤4

.bablerc配置文件

```
{
    "presets": [
        "es2015"
    ],
    "plugins": [
        "transform-runtime"
    ]
}
复制代码
```

#### 步骤5

入口文件app.js

```
import Vue from 'vue';

import app from './app/App.vue';

new Vue({
    el: '#app',
    render: h => h(app)
})
复制代码
```

#### 步骤6

./app/App.vue

```vue
<template>
  <div class="content">
    <header class="header">My Todo List</header>
    <add-todo @add="add"></add-todo>
    <todo-list :todos="todos"></todo-list>
  </div>
</template>

<script>
import Vue from "vue";

import AddTodo from "./components/AddTodo.vue";
import TodoList from "./components/TodoList.vue";

export default {
  components: {
    AddTodo,
    TodoList
  },
  data() {
    return {
      todos: [
        { name: "JavaScript", status: true },
        { name: "Css", status: false },
        { name: "Vue", status: true }
      ],
      bus: new Vue()
    };
  },
  methods: {
    add(todo) {
      this.todos.push({ name: todo, status: false });
    }
  }
};
</script>

<style scoped>
.content {
  width: 800px;
  height: 600px;
  margin: 0 auto;
  background-color: rgb(245, 245, 245);
  padding: 24px 0;
}

.header {
  font-size: 48px;
  text-align: center;
}
</style>
```

./app/components/TodoList.vue

```vue
<template>
  <div class="content-div">
    <p class="content-div">
      <input type="radio" v-model="picked" value="0" @click="filterTodos(0)" />
      <label>所有</label>
      <input type="radio" v-model="picked" value="1" @click="filterTodos(1)" />
      <label>已完成</label>
      <input type="radio" v-model="picked" value="2" @click="filterTodos(2)" />
      <label>未完成</label>
    </p>
    <p
      class="content-div"
    >当前共有{{todoList.length}}个代办事项,已完成{{completed}}个,剩余{{todoList.length-completed}}个。</p>
    <ul class="list">
      <li class="item" v-for="(todo,index) in tempTodoList" :key="index">
        <div>
          <span class="item">{{index+1}}. {{todo.name}}</span>
          <button v-if="todo.status" @click="changeStatus(index)">已完成</button>
          <button v-else @click="changeStatus(index)">未完成</button>
        </div>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  props: ["todos"],
  data: function() {
    return {
      todoList: this.todos,
      picked: 0,
      tempTodoList: []
    };
  },
  computed: {
    completed() {
      return this.todoList.filter(x => x.status).length;
    }
  },
  methods: {
    changeStatus(index) {
      this.tempTodoList[index].status = !this.tempTodoList[index].status;
      this.filterTodos(parseInt(this.picked));
    },
    filterTodos(type) {
      this.$root.$emit("selectChanged", type);
      switch (type) {
        case 0:
          this.tempTodoList = this.todoList;
          break;
        case 1:
          this.tempTodoList = this.todoList.filter(x => x.status);
          break;
        case 2:
          this.tempTodoList = this.todoList.filter(x => !x.status);
          break;
      }
    }
  },
  mounted() {
    this.tempTodoList = this.todoList;
  }
};
</script>

<style scoped>
.content-div {
  font-size: 16px;
  margin-left: 15px;
}
.list {
  list-style: none;
  padding: 0px;
  margin-left: 15px;
}
.item {
  list-style: none;
  display: inline-block;
  width: 90%;
  padding-top: 10px;
}
</style>
```

./app/components/AddTodo.vue

```vue
<template>
  <div class="add-content">
    <input type="text" class="text_input" placeholder="请输入代办事项名称" v-model.trim="todo" />
    <button :disabled="canAddTodo" @click="add" class="add_button">添加</button>
  </div>
</template>

<script>
export default {
  created() {
    this.$root.$on("selectChanged", value => {
      this.canAddTodo = !!value;
    });
  },
  data() {
    return {
      todo: "",
      canAddTodo: false
    };
  },
  methods: {
    add() {
      this.$emit("add", this.todo);
      this.todo = "";
    }
  }
};
</script>

<style scoped>
.add-content {
  font-size: 16px;
  margin-left: 15px;
  padding: 10px;
}
.text_input {
  width: 90%;
  height: 30px;
  padding: 10px 10px;
  font-size: 14px;
  border: 1px solid transparent;
}
.add_button {
  width: 60px;
  height: 30px;
}
</style>
```

## vue-router

> 本章节会在上一节**工程化**的基础上，对TodoList进行修改。

### 目录结构



![img](五个实例帮你快速入门Vue.assets/170499ea06af07b6)



### 页面截图



![img](五个实例帮你快速入门Vue.assets/170499f917183986)



### 关键知识

1. vue-router的基础使用;
2. 使用浏览器缓存保存数据;
3. 思考这样的方式有何缺点?

### 完整代码

#### 步骤1

```
npm i vue-router -S
复制代码
```

#### 步骤2

修改webpack.config.js

```
const path = require('path');
const VueLoaderPlugin = require('vue-loader/lib/plugin');

const config = {
    ...
    module: {
        rules: [
            {
                test: /\.vue$/,
                use: 'vue-loader',
            }
            ...
        ]
    },
    plugins: [
        new VueLoaderPlugin()
    ]
}

复制代码
```

#### 步骤3

app.js

```
import Vue from 'vue';

import VueRouter from 'vue-router';

import App from './App.vue'
import Index from './views/All.vue';
import Completed from './views/Completed.vue';
import Uncompleted from './views/UnCompleted.vue';

Vue.use(VueRouter);


const routerPush = VueRouter.prototype.push;
VueRouter.prototype.push = function push(location) {
    return routerPush.call(this, location).catch(error => error)
}

const routes = [
    {
        path: '/',
        redirect: '/index'
    }, {
        path: '/index',
        name: 'all',
        component: Index
    }, {
        path: '/completed',
        name: 'completed',
        component: Completed
    }, {
        path: '/uncompleted',
        name: 'uncompleted',
        component: Uncompleted
    }
];

const router = new VueRouter({
    routes
})

new Vue({
    el: '#app',
    router,
    render: h => h(App)
})
复制代码
```

#### 步骤4

App.vue

```
<template>
  <div class="content">
    <header class="header">My Todo List</header>
    <div class="add-content">
      <input type="text" class="text_input" placeholder="请输入代办事项名称" v-model.trim="todo" />
      <button @click="add" class="add_button">添加</button>
      <p
        class="content-div"
      >当前共有{{todoList.length}}个代办事项,已完成{{completed}}个,剩余{{todoList.length-completed}}个。</p>
      <p class="content-div">
        
        <router-link to="/index">全部</router-link>
        <router-link to="/completed">已完成</router-link>
        <router-link to="/uncompleted">未完成</router-link>
      </p>
    </div>

    <router-view ></router-view>
  </div>
</template>

<script>
export default {
  data() {
    return {
      todo: "",
      todoList: [],
      picked: 0,
    };
  },
  computed: {
    completed() {
      this.todoList = JSON.parse(window.localStorage.getItem("todos"));
      return this.todoList.filter(x => x.status).length;
    }
  },
  methods: {
    add() {
      this.picked = 0;
      this.todoList.push({ name: this.todo, status: false });
      window.localStorage.setItem("todos", JSON.stringify(this.todoList));
      this.$router.go(0);
    }
  },
  mounted() {
    this.todoList = [
      { name: "JavaScript", status: true },
      { name: "Css", status: false },
      { name: "Vue", status: true }
    ];

     window.localStorage.setItem("todos", JSON.stringify(this.todoList));
  }
};
</script>

<style scoped>
.content {
  width: 800px;
  height: 600px;
  margin: 0 auto;
  background-color: rgb(245, 245, 245);
  padding: 24px 0;
}
.header {
  font-size: 48px;
  text-align: center;
}
.content-div {
  font-size: 16px;
}
.add-content {
  font-size: 16px;
  margin-left: 15px;
  padding: 10px;
}
.text_input {
  width: 90%;
  height: 30px;
  padding: 10px 10px;
  font-size: 14px;
  border: 1px solid transparent;
}
.add_button {
  width: 60px;
  height: 30px;
}
</style>
复制代码
```

#### 步骤5

> All.vue、 Completed.vue以及Uncompleted.vue代码除了计算属性那块有点却别之外，其它基本相同。为了演示路由的使用，所以将其分为三个不同的页面。

```
<template>
  <ul class="list">
    <li class="item" v-for="(todo,index) in todos" :key="index">
      <div>
        <span class="item">{{index+1}}. {{todo.name}}</span>
        <button v-if="todo.status" @click="changeStatus(todo.name)">已完成</button>
        <button v-else @click="changeStatus(todo.name)">未完成</button>
      </div>
    </li>
  </ul>
</template>

<script>
export default {
  computed: {
    todos() {
      return JSON.parse(window.localStorage.getItem("todos"));
    }
  },
  methods: {
    changeStatus(name) {
      const todos = JSON.parse(window.localStorage.getItem("todos"));
      todos.forEach(todo => {
        if(todo.name === name){
          todo.status = !todo.status;
        }
      });
       window.localStorage.setItem("todos", JSON.stringify(todos));
       this.$router.go(0);
    }
  }
};
</script>

<style  scoped>
.list {
  list-style: none;
  padding: 0px;
 margin: 0px 25px;
}

.item {
  list-style: none;
  display: inline-block;
  width: 90%;
  padding: 5px 0px;
}
</style>
```

Completed.vue和Uncompleted.vue文件的computed

```js
 computed: {
    todos() {
      const todoList = JSON.parse(window.localStorage.getItem("todos"));
      return todoList.filter(x => x.status);
    }
  },
```

## vuex

### 目录结构

![img](五个实例帮你快速入门Vue.assets/17049b07c29cc4e4)

### 关键知识

1. 如何使用vuex;
2. 如何获取状态中的属性；
3. 如何同步更改状态;

### 完整代码

#### 步骤1

```
 npm i vuex -S
```

#### 步骤2

store.js

```
import Vue from 'vue';
import Vuex from 'vuex';

Vue.use(Vuex);

const store = new Vuex.Store({
    state: {
        todos: [
            { name: "JavaScript", status: true },
            { name: "Css", status: false },
            { name: "Vue", status: true }
        ]
    },
    mutations: {
        changeTodoState(state, name) {
            state.todos.forEach(todo => {
                if (todo.name === name) {
                    todo.status = !todo.status;
                }
            });
        },
        addTodo(state, todo) {
            state.todos.push(todo);
        }
    }
});

export default store;
```

#### 步骤3

修改app.js

```
.....
import store from './store.js';
.....
new Vue({
    el: '#app',
    store,
    router,
    render: h => h(App)
})
```

修改App.vue

```
<template>
.....
   <p
        class="content-div"
      >当前共有{{this.$store.state.todos.length}}个代办事项,已完成{{completed}}个,剩余{{this.$store.state.todos.length-completed}}个。
    </p>
.....
</template>

<script>
export default {
  ......
  computed: {
    completed() {
      return this.$store.state.todos.filter(x => x.status).length;
    }
  },
  methods: {
    add() {
      this.$store.commit("addTodo", { name: this.todo, status: false });
    }
  }
};
</script>
```

修改All.vue、Completed.vue、Uncompleted.vue

```
......
<script>
export default {
  computed: {
    todos() {
      return this.$store.state.todos.filter(x => x.status);
    }
  },
  methods: {
    changeStatus(name) {
      this.$store.commit('changeTodoState',name);
    }
  }
};
</script>
......
```

## 总结

本文演示的例子比较简单，所以也没有做太多的讲解。动手来实现一遍远比看十遍的文档也有用。

都到这儿了，如果本文对你有所帮助，麻烦给个❤。

欢迎交流~