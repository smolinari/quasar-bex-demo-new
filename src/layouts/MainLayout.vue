<template>

  <q-layout view="lHh Lpr lFf">
    <q-drawer
      v-model="uiVisible"
      content-class="bg-grey-2"
    >
      <q-list>
        <q-item-label header>Your Quasar To Do List</q-item-label>

        <div class="bg-white q-ma-md q-pa-md">
          <q-input v-model="todo.text" label="Add a todo" />
          <q-btn @click="addTodo" color="primary" class="full-width q-mt-md">Add</q-btn>

          <q-list
            class="q-mt-md"
          >
            <q-item
              v-for="todo in todos"
              :key="todo.id"
            >
              <q-item-section>
                <a :href="todo.link" target="_blank">{{todo.text}}</a>
              </q-item-section>
              <q-item-section side>
                <q-btn
                  icon="remove"
                  color="primary"
                  size="xs"
                  round
                  @click="removeTodo(todo.id)"
                >
                </q-btn>
              </q-item-section>
            </q-item>
          </q-list>
        </div>
      </q-list>

      <div class="flex flex-center">
        <q-btn flat dense @click="toggleToolbar(null, false)" icon="close">Close Todo List</q-btn>
      </div>
    </q-drawer>
     <q-page-container>
       <!-- <router-view /> -->
     </q-page-container>
  </q-layout>

</template>

<script>
import dbService from 'src/services/db'
import { uid } from 'quasar'

export default {
  name: 'MyLayout',

  data () {
    return {
      uiVisible: false,
      todo: {
        id: null,
        text: '',
        type: 'todo',
        link: '',
        status: 0
      },
      todos: []
    }
  },

  methods: {
    toggleToolbar (payload, onlyOpen = false) {
      if (onlyOpen === false || (onlyOpen && this.uiVisible === false)) {
        this.uiVisible = !this.uiVisible
        return this.$q.bex.send('bex.toggle.iframe', {
          open: this.uiVisible
        })
      }
    },

    resetTodo () {
      this.todo.id = uid()
      this.todo.text = ''
      this.todo.link = ''
      this.todo.status = 0
    },

    loadTodos () {
      dbService.getAll('todo').then(todos => {
        this.todos = todos
      })
    },

    addTodo () {
      dbService.save(`todo.${this.todo.id}`, this.todo)
      this.resetTodo()
      this.loadTodos()
    },

    removeTodo (id) {
      console.log('deleting', id)
      dbService.delete('todo', id)
      this.resetTodo()
      this.loadTodos()
    },

    addRemoteTodo (payload) {
      const data = payload.data
      this.todo.id = uid()
      this.todo.text = data.text
      this.todo.link = data.link
      this.addTodo()

      // once the toolbar has been opened, notify the user.
      this.toggleToolbar(null, true)
      this.$q.notify({
        message: 'Todo has been added for ' + data.text,
        color: 'positive'
      })
    }
  },

  created () {
    this.$q.bex.on('bex.toggle.toolbar', this.toggleToolbar)
    this.$q.bex.on('bex.add.todo', this.addRemoteTodo)

    this.loadTodos()
  },

  beforeDestroy () {
    this.$q.bex.off('bex.toggle.toolbar', this.toggleToolbar)
    this.$q.bex.off('bex.add.todo', this.addRemoteTodo)
  }
}
</script>
