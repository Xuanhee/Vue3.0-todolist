<template>
  <section id="app" class="todoapp">
    <header class="header">
      <h1>todos</h1>
      <input
        class="new-todo"
        placeholder="What needs to be done?"
        autocomplete="off"
        autofocus
        v-model="input"
        @keyup.enter="addtodos"
        >
    </header>
    <section class="main" v-show="count">
      <input id="toggle-all" class="toggle-all" type="checkbox" v-model="allDone">
      <label for="toggle-all" >Mark all as complete</label>
      <ul class="todo-list">
        <li
        v-for="todo in filterTodos"
        :key = todo
        :class="{ editing: todo === editingTodo, completed: todo.completed}"
        >
          <div class="view">
            <input class="toggle" type="checkbox" v-model="todo.completed">
            <label @dblclick="editTodo(todo)">{{ todo.text }}</label>
            <button class="destroy" @click="remove(todo)"></button>
          </div>
          <input
            class="edit"
            type="text"
            v-model="todo.text"
            v-editing-focus="todo === editingTodo"
            @keyup.enter="enterEdit(todo)"
            @blur="enterEdit(todo)"
            @keyup.esc="cancelEdit(todo)"
            >
        </li>
      </ul>
    </section>
    <footer class="footer" v-show="count">
      <span class="todo-count">
        <strong>{{ remainingCount }}</strong> {{ remainingCount > 1 ? 'items' : 'item'}} left
      </span>
      <ul class="filters">
        <li><a href="#/all">All</a></li>
        <li><a href="#/active">Active</a></li>
        <li><a href="#/completed">Completed</a></li>
      </ul>
      <button class="clear-completed" @click="removeCompleted" v-show="count > remainingCount">
        Clear completed
      </button>
    </footer>
  </section>
</template>  

<script>
import { computed, onMounted, onUnmounted, ref, watchEffect } from 'vue'
import useLocalStorage from './utils/useLocalStorage'
import './assets/index.css'

const storage = useLocalStorage()

// 1.添加待办事项
const useAdd = todos => {
  // 文本框输入内容
  const input = ref('')
  // 点击回车保存待办事项
  const addtodos = () => {
    const text = input.value && input.value.trim()
    if(text.length === 0) return
    todos.value.unshift({
      text,
      completed: false
    })
    // 保存完毕之后清空文本框内容
    input.value = ''
  }
  return {
    input,
    addtodos
  }
}

// 2. 删除待办事项
const useRemove = todos => {
  // 点击删除按钮删除事件
  const remove = todo => {
    // 找到todo在todos中的索引
    const index = todos.value.indexOf(todo)
    todos.value.splice(index, 1)
  }
  // 删除已完成项  
  const removeCompleted = () => {
    todos.value = todos.value.filter(todo => !todo.completed)
  }
  return {
    remove,
    removeCompleted
  }
}

// 3.编辑待办项
const useEdit = remove => {
  // 保存修改前的数据
  let beforeEditingText = ''
  // 修改的数据
  const editingTodo = ref(null)

  // 双击
  const editTodo = todo => {
    beforeEditingText = todo.text
    // 保存当前项
    editingTodo.value = todo
  }
  // 回车保存
  const enterEdit = todo => {
    if(!editingTodo.value) return
    todo.text = todo.text.trim()
    todo.text || remove(todo)
    editingTodo.value = null
  }
  // 取消
  const cancelEdit = todo => {
    editingTodo.value = null
    todo.text = beforeEditingText
  }

  return {
    editingTodo,
    editTodo,
    enterEdit,
    cancelEdit
  }
}

// 4.切换待办项完成状态
const useFilter = todos => {
  const allDone = computed({
    get() {
      // 筛选未完成项的数量，然后取反来控制所有完成还是存在未完成
      return !todos.value.filter(todo => !todo.completed).length
    },
    set(value) {
      todos.value.forEach(todo => {
        todo.completed = value
      });
    }
  })

  const filter = {
    all: todos => todos,
    active: todos => todos.filter(todo => !todo.completed),
    completed: todos => todos.filter(todo => todo.completed)
  }
  // 响应式保存hash值
  const type = ref('all')
  // 筛选当前hash渲染的列表
  const filterTodos = computed(() => filter[type.value](todos.value))
  // 未完成的待办事项的数量
  const remainingCount = computed(() => filter.active(todos.value).length)
  // todos总数
  const count = computed(() => todos.value.length)
  // hash改变事件
  const onHashChange = () => {
    const hash = window.location.hash.replace('#/', '')
    if(filter[hash]){
      type.value = hash
    }else {
      type.value = 'all'
      window.location.hash = ''
    }
  }

  onMounted(() => {
    window.addEventListener('hashchange', onHashChange)
    onHashChange()
  })

  onUnmounted(() => {
    window.removeEventListener('hashchange', onHashChange)
  })

  return {
    allDone,
    filterTodos,
    remainingCount,
    count
  }
}

// 5.存储待办事项
const useStorage = () => {
  const KEY = 'TODOKEYS'
  const todos = ref(storage.getItem(KEY) || [])
  watchEffect(() => {
    storage.setItem(KEY, todos.value)
  })
  return todos
}

export default {
  name: 'App',
  setup() {
    const todos = useStorage()
    const { input, addtodos} = useAdd(todos)
    const { remove, removeCompleted } = useRemove(todos)
    const { editingTodo, editTodo, enterEdit, cancelEdit } = useEdit(remove)
    const { allDone, filterTodos, remainingCount, count } = useFilter(todos)
    // 返回的成员在模板或组件其他位置都可以使用
    return {
      input,
      addtodos,
      remove,
      removeCompleted,
      todos,
      editingTodo,
      editTodo,
      enterEdit,
      cancelEdit,
      allDone,
      filterTodos,
      remainingCount,
      count
    }
  },
  // 自定义指令
  directives: {
    editingFocus: (el, binding) => {
      binding.value && el.focus()
    }
  }
}

</script>

<style>
</style>
