<template>
<div id="home">
  <div class="container-fluid mask w-100 h-100">
    <div class="row w-100 h-100">
      <div class="allpage" :class="{openlist: list || setting}">
        <div class="clock">
          <div class="top">
            <div>
              <div class="showtime d-inline-block" style="width:85%">
                <p class="text-center">{{today}}</p>
                <p class="text-center">{{hrs}}:{{min}}</p>
              </div>
              <div class="icon_right float-right d-flex align-items-center flex-column mt-4" style="width:15%">
                <i class="fas fa-tasks" @click="changelist" :class="{active:list}"></i>
                <i class="fas fa-cog mt-3"  @click="changesetting" :class="{active:setting}"></i>
              </div>
            </div>
          </div>
          <div class="mainclock container-fluid m-0 d-inline-block text-center mt-3">
            <div class="timer d-inline-block text-center">
              <div class="col-12">{{ timetext }}</div>
              <div class="col-12 icon_bottom">
                <div class="icon" v-if="status !== 1" @click="start">
                  <i class="fas fa-play"></i>
                </div>
                <div class="icon" v-if="status === 1" @click="pause">
                  <i class="fas fa-pause"></i>
                </div>
                <div class="icon" v-if="current.length > 0" @click="finish(true)">
                  <i class="fas fa-step-forward"></i>
                </div>
              </div>
            </div>
            <div class="dothing mt-1">
              <p class="text-center">{{currentText}}</p>
            </div>
          </div>
        </div>
        <div class="list" :class="{moveleft:list}">
          <div class="list_top">
            <input type="text" placeholder="請輸入代辦事項" v-model="newtodo" trim :state="state" @keydown.enter="addTodo">
            <i class="fas fa-plus-circle addbtn" @click="addTodo"></i>
          </div>
          <div class="container-fluid p-0 mt-5">
            <div class="row list_item mb-4 border-bottom" v-for="(item,index) in todos" :key="index">
              <div class="col-9">
                <input type="text" v-if="item.edit"
                v-model="item.model"
                trim
                :state="stateTodo(item.model)"
                @keydown.enter="changeTodo(index)">
                <span v-else>{{item.name}}</span>
              </div>
              <div class="col-3 text-right">
                <span v-if="item.edit">
                  <i class="fas fa-check mr-3" @click="changeTodo(index)"></i>
                  <i class="fas fa-undo" @click="cancelTodo(index)"></i>
                </span>
                <span v-else>
                  <i class="fas fa-pen mr-3" @click="editTodo(index)"></i>
                  <i class="fas fa-times" @click="delTodo(index)"></i>
                </span>
              </div>
            </div>
          </div>
        </div>
        <div class="musicset" :class="{moveleft:setting}">
          <div class="container-fluid">
            <div class="row">
              <div class="col-11 ml-auto">
                <b-table :items='items' :fields="fields" @row-clicked="slectAlarm">
                  <template #cell(file)="data">
                    <audio controls :src="'./sounds/'+data.item.file"></audio>
                  </template>
                  <template #cell(select)="data">
                    <i class="fas fa-check" v-if="data.item.file === alarm"></i>
                  </template>
                </b-table>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>
</template>

<script>
export default {
  name: 'Home',
  data () {
    return {
      timer: 0,
      now: '',
      today: '',
      hrs: '',
      min: '',
      list: false,
      setting: false,
      newtodo: '',
      items: [
        {
          name: '鬧鈴',
          file: 'alarm1.mp3'
        },
        {
          name: 'yay',
          file: 'yay.mp3'
        }
      ],
      fields: [
        {
          key: 'name',
          label: '名稱'
        },
        {
          key: 'file',
          label: '預覽'
        },
        {
          key: 'select',
          label: '選擇'
        }
      ]
    }
  },
  computed: {
    status: {
      get () {
        return this.$store.state.status
      },
      set (value) {
        this.$store.commit('changeStatus', value)
      }
    },
    state () {
      if (this.newtodo.length === 0) {
        return null
      } else if (this.newtodo.length < 2) {
        return false
      } else {
        return true
      }
    },
    current () {
      return this.$store.state.current
    },
    currentText () {
      let result = ''

      if (this.current.length > 0) {
        result = this.current
      } else if (this.todos.length > 0) {
        result = '點擊開始'
      } else {
        result = '沒有事項'
      }

      return result
    },
    timeleft () {
      return this.$store.state.timeleft
    },
    alarm () {
      return this.$store.state.alarm
    },
    todos () {
      return this.$store.state.todos
    },
    timetext () {
      const m = Math.floor(this.timeleft / 60)
      const s = Math.floor(this.timeleft % 60)
      return `${m} : ${s}`
    }
  },
  methods: {
    addTodo () {
      if (this.state) {
        this.$store.commit('addTodo', this.newtodo)
        this.newtodo = ''
      }
    },
    delTodo (index) {
      this.$store.commit('delTodo', index)
    },
    editTodo (index) {
      this.$store.commit('editTodo', index)
    },
    stateTodo (data) {
      if (data.length < 2) {
        return false
      } else {
        return true
      }
    },
    changeTodo (index) {
      const valid = this.stateTodo(this.todos[index].model)

      if (valid) {
        this.$store.commit('changeTodo', index)
      }
    },
    cancelTodo (index) {
      this.$store.commit('cancelTodo', index)
    },
    changelist () {
      if (this.setting === true) {
        this.setting = false
      }
      this.list = !this.list
    },
    changesetting () {
      if (this.list === true) {
        this.list = false
      }
      this.setting = !this.setting
    },
    start () {
      // 不是暫停繼續，且待辦有東西時才把 todos 的第一個變成 current
      if (this.status !== 2 && this.todos.length > 0) {
        this.$store.commit('start')
      }
      if (this.current.length > 0) {
        this.status = 1
        this.timer = setInterval(() => {
          this.$store.commit('countdown')
          if (this.timeleft <= -1) {
            this.finish(false)
          }
        }, 1000)
      }
    },
    finish (skip) {
      clearInterval(this.timer)
      this.status = 0
      this.$store.commit('finish')

      if (!skip) {
        const audio = new Audio()
        audio.src = './sounds/' + this.alarm
        audio.play()
      }

      if (this.todos.length > 0) {
        this.start()
      } else {
        alert('全部結束')
      }
    },
    pause () {
      clearInterval(this.timer)
      this.status = 2
    },
    renderTime () {
      setInterval(() => {
        this.now = new Date()
        this.today = this.now.toDateString()
        this.time = this.now.toLocaleTimeString()
        this.hrs = this.now.getHours()
        this.min = this.now.getMinutes()
        if (this.min < 10) {
          this.min = '0' + this.min
        }
      }, 1000)
    },
    slectAlarm (item) {
      this.$store.commit('selectAlarm', item.file)
    }
  },
  mounted () {
    this.renderTime()
  }
}
</script>

<style lang="stylus" scoped>
::-webkit-scrollbar {
  width 10px
}
::-webkit-scrollbar-track {
  background-color transparent
}
::-webkit-scrollbar-thumb {
  background-color white
  border-radius 5px
}
#home{
  background: url('../assets/1.jpg') no-repeat center center/cover
  width 100vw
  height 100vh
  z-index -10
}
.mask{
  background-color rgba(255,255,255,0.5)
  z-index -9
}
.allpage{
  position absolute
  top 50%
  left 50%
  transform translate(-50%,-50%)
  z-index 99
  transition 0.5s ease
}
.openlist{
  left 30%
}
.list{
  background-color rgba(0,0,0,0.5)
  width 24vw
  min-width 350px
  height 65vh
  position absolute
  top 50%
  left 50%
  z-index -1
  transform translate(-50%,-50%)
  backdrop-filter blur(10px)
  transition 0.5s ease
  overflow auto
  &_top{
    input{
      margin-top 50px
      margin-left 80px
      width 70%
      border-top none
      border-right none
      border-left none
      border-bottom 1px solid white
      padding 5px 20px
      text-align left
      background-color rgba(255, 255, 255, 0)
      color rgba(255, 255, 255, 1)
      font-size 24px
    }
    input:focus{
      border-color transparent
      outline none
      border-bottom 1px solid white
    }
    input::placeholder {
      font-size 24px
      text-align left
      color rgb(255, 255, 255)
    }
    .addbtn{
      margin-left 20px
      font-size 2.5rem
      color white
      transition 0.1s ease
      &:active{
        transform scale(1.2)
        filter: drop-shadow(0px 0px 5px rgba(255, 255, 255, 0.5))
      }
    }
  }
  &_item{
    margin-left 80px
    width 70%
    color white
    input{
      width 70%
      border none
      text-align left
      background-color rgba(255, 255, 255, 0)
      color white
      text-shadow 0 0 5px rgba(255, 255, 255, 1)
      font-size 24px
    }
    input:focus{
      border-color transparent
      outline none
    }
  }
}
.musicset{
  background-color rgba(0,0,0,0.5)
  width 24vw
  min-width 350px
  height 65vh
  position absolute
  top 50%
  left 50%
  z-index -1
  transform translate(-50%,-50%)
  backdrop-filter blur(10px)
  transition 0.5s ease
  overflow auto
  table{
    color white
    audio{
      width 250px
      height 30px
    }
  }
}
.moveleft{
  width 45vw
  position absolute
  top 50%
  left 50%
  transform translate(25%,-50%)
}
.clock{
  width 25vw
  min-width 350px
  height 70vh
  border-radius 20px
  background: rgba(255,255,255,1)
  backdrop-filter blur(5px)
  box-shadow: 0 0 10px 0px rgba(0,0,0,0.5),0 0 30px 5px rgba(0,0,0,0.5)
  z-index 10
  position relative
  .top{
    width 100%
    height 120px
    background rgba(150,150,250,0.8)
    border-radius 20px 20px 0 0
    font-size 2rem
    color white
    .hrsmin{
      font-size 2.5rem
    }
  }
  .icon_right i{
    font-size 2rem
    transition 0.3s ease
    &:active{
      transform scale(1.2)
    }
    &:hover{
      color rgba(100,100,150,1)
    }
  }
}
.active{
  color rgba(50,50,150,1)
}
.timer{
  width 200px
  height 200px
  padding 20px
  font-size 2.5rem
  border 5px solid black
  border-radius 50%
  .icon_bottom{
    font-size 2rem
  }
}
.dothing p{
  font-size 2rem
}
.mainclock{
  width 100%
}
</style>
