<template>
  <div class="container">
    <div>
      <h1></h1>
      <h3 style="color:#666666">新しいToDoを作成する</h3>
      <el-form label-position="left" label-width="80px" class="input-form">
        <el-form-item label="ToDo名">
          <el-input type="text" v-model="todoTitle"/>
        </el-form-item>

        <el-form-item label="期限">
          <el-date-picker
            v-model="todoDDL"
            type="date"
            placeholder="Pick a day"
            class="datepicker">
          </el-date-picker>
          <el-button type="primary" class="addtodo-button" v-on:click="sendForm" icon="el-icon-plus">ToDoの追加</el-button>
        </el-form-item>
        <div class="alert-todo">
        <el-alert class="alert-todo-inner" v-show="isAlertShowDDL" title="期限は作成日の後にしてください" type="error" show-icon></el-alert>
        <el-alert class="alert-todo-inner" v-show="isAlertShowDDL2" title="期限を選択してください" type="error" show-icon></el-alert>
        <el-alert class="alert-todo-inner" v-show="isAlertShowNULL" title="ToDo名を必ず入力してください" type="error" show-icon></el-alert>
        <el-alert class="alert-todo-inner" v-show="isAlertShowOver" title="ToDo名を３０字以内にしてください" type="error" show-icon></el-alert>
        <el-alert class="alert-todo-inner" v-show="isAlertShowSame" title="入力したToDo名はすでに存在しています" type="error" show-icon></el-alert>
        </div>

      </el-form>
    </div>

    <div>
      <div v-if="list.length !== 0">
        <div v-for="(item,index) in list" v-bind:key="index"  class="todo-item">
          <el-card class="box-card" shadow="hover">
            <div slot="header" class="clearfix">
              <span>{{item.todoTitle}}</span>
            </div>
            <div class="todo-button" style="width:200px;float:right">
            <el-button type="success" round v-on:click="jumpToedit(item)" style="float:left">編集</el-button>
            <el-button :type="item.isDone?'primary':'danger'" round v-on:click="changeButton(item)" style="float:right">{{item.isDone?'完了':'未完了'}}</el-button>
            </div>
            <div class="text item">
              <p>期限：{{getDate(item.todoDDL)}}</p>
              <p>作成日：{{getDate(item.todoCreateData)}}</p>
            </div>
          </el-card>
        </div>
      </div>
      <div v-else>
        <el-alert title="登録されたToDoはございません" type="error"></el-alert>
      </div>
    </div>
  </div>
</template>

<script>
import moment from 'moment'
import _ from 'lodash'
import axios from 'axios'

export default {
  data () {
    return {
      // lists: [],
      list: [],
      // TodoItem: [],
      todoTitle: '',
      todoDDL: '',
      todoCreateData: '',
      isAlertShowDDL: false,
      isAlertShowDDL2: false,
      isAlertShowNULL: false,
      isAlertShowOver: false,
      isAlertShowSame: false,
      buttonType: 'danger',
      buttonText: '未完了',
      type: ''
    }
  },
  created () {
    this.getAllTodoitem()
    this.getAllTodoitem()
  },
  methods: {
    getTitle () {
      // this.list = this.lists[this.$route.query.id]
      return this.list.title
    },
    sendForm () {
      var _this = this
      var checkTodo = false
      var checkDDL = false
      if (this.todoTitle.length === 0) {
        // ToDo名は空の時
        // console.log(this.TodoTitle.length)
        this.isAlertShowNULL = true
      } else {
        this.isAlertShowNULL = false
        if (this.todoTitle.length >= 31) {
          // ToDo名は３１文字以上の時
          this.isAlertShowOver = true
        } else {
          this.isAlertShowOver = false
        }
        if (this.isInArray(this.list, this.todoTitle)) {
          // ToDo名はすでに存在している時
          this.isAlertShowSame = true
        } else {
          this.isAlertShowSame = false
        }
      }
      checkTodo = (!this.isAlertShowNULL) && (!this.isAlertShowOver) && (!this.isAlertShowSame)
      if (this.todoDDL === '') {
        // 期限は空の時
        this.isAlertShowDDL2 = true
      } else {
        this.isAlertShowDDL2 = false
        var today = new Date()
        if (moment(this.todoDDL).isBefore(today) || moment(this.todoDDL).isSame(today)) {
          // 期限は作成日の前
          this.isAlertShowDDL = true
        } else {
          // 期限は作成日の後
          this.isAlertShowDDL = false
        }
      }
      // console.log(this.$route.params.title)
      checkDDL = (!this.isAlertShowDDL2) && (!this.isAlertShowDDL)
      if (checkTodo && checkDDL) {
        var item = {
          // user: this.$route.params.user,
          // _id: this.list._id,
          // title: this.$route.params.title,
          todoTitle: this.todoTitle,
          todoDDL: this.todoDDL,
          todoCreateData: new Date(),
          isDone: false}
        console.log(item)
        // this.list.TodoItem.unshift(item)
        axios.post('/api/createTodoitem', item)
          .then(function (response) {
            console.log(item)
            console.log('success')
            // console.log(item)
            console.log(response)
            _this.getAllTodoitem()
          })
          .catch(function (error) {
            console.log(error)
          })
        // this.getAllTodoitem()
        // this.getAllTodoitem()

        this.todoDDL = ''
        this.todoTitle = ''
        this.todoCreateData = ''
      }
    },
    isInArray (list, keyword) {
      // var TodoItem = list.TodoItem
      return _.find(list, {todoTitle: keyword})
    },
    getDate (item) {
      return moment(item).format('LL')
    },
    changeButton (item) {
      item.isDone = !item.isDone
      // console.log(item.isDone)
      // console.log(item.isDone)
      var change = {
        // _id: item._id,
        // TodoTitle: item.TodoTitle,
        id: item.id,
        todoTitle: item.todoTitle,
        todoDDL: item.todoDDL,
        todoCreateData: item.todoCreateData,
        isDone: item.isDone
        // user: this.$route.params.user
      }
      axios.post('/api/changeIsdone', change)
        .then(function (response) {
          console.log(response.data)
        })
        .catch(function (error) {
          console.log(error)
        })
      // this.getAllTodoitem()
      // this.getAllTodoitem()
    },
    htmlEscape (str) {
      if (!str) return
      return str.replace(/[<>&"'`]/g, function (match) {
        const escape = {
          '<': '&lt;',
          '>': '&gt;',
          '&': '&amp;',
          '"': '&quot;',
          "'": '&#39;',
          '`': '&#x60;'
        }
        return escape[match]
      })
    },
    getAllTodoitem () {
      var _this = this

      // var item = {id: 0}
      // console.log(_this.$route.meta)
      axios.get('/api/getallTodoitem')
        .then(function (response) {
          _this.list = response.data
          // console.log('success')
          console.log(response.data)
        })
        .catch(function (error) {
          console.log(error)
        })
    },
    jumpToedit (item) {
      // console.log(key)
      this.$router.push({
        name: 'Edit',
        params: {
          id: item.id
        }
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.container {
  margin: 0 auto;
  width: 70%
}
.text {
  font-size: 10px;
  color:#a5a5a5
}
.todo-item {
  margin-bottom: 20px;
    .el-card__body {
      padding-bottom: 5px;
      padding-top: 5px
    }
}
.item {
    margin-bottom: 0px;
  }
.input-form {
  width: 70%
}
.addtodo-button {
  float: right
}
.container {
  .el-date-editor {
    width: 70%
  }
}
.alert-list {
  margin-bottom: 20px
}
.alert-todo-inner {
  margin-bottom:10px
}
</style>
