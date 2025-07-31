<script setup>
import { inject, ref, reactive, onMounted } from "vue"
import socketManager from '../socketManager.js'

// #region global state
const userName = inject("userName")
// #endregion

// #region local variable
const socket = socketManager.getInstance()
// #endregion
const minDateTime = ref(new Date().toISOString().slice(0, 16))

function getJstDatetimeLocal() {
  const now = new Date()
  now.setSeconds(0)
  now.setMilliseconds(0)

  const year = now.getFullYear()
  const month = String(now.getMonth() + 1).padStart(2, '0')
  const day = String(now.getDate()).padStart(2, '0')
  const hour = String(now.getHours()).padStart(2, '0')
  const minute = String(now.getMinutes()).padStart(2, '0')

  return `${year}-${month}-${day}T${hour}:${minute}`
}

const jstDatetime = getJstDatetimeLocal()
console.log(jstDatetime) // 例: 2025-07-31T16:13

// #region reactive variable
const chatContent = ref("")
const taskContent = ref("")
const chatList = reactive([])

const toWho = ref("")

const selectedDate = ref("")

// タスクリスト用の変数を追加
const taskList = reactive([
  { who: "田中さん", when: "2025/07/31 10:00", what: "資料作成" },
  { who: "佐藤さん", when: "2025/07/31 14:30", what: "会議準備" },
  { who: "鈴木さん", when: "2025/08/01 09:00", what: "レビュー実施" },
  { who: "", when: "2025/08/01 09:00", what: "レビュー実施" },
  { who: "小林さん", when: "2025/06/31 10:00", what: "資料作成" },
  { who: "植木さん", when: "2025/07/15 08:00", what: "会議準備" }
])

// #endregion

// #region lifecycle
onMounted(() => {
  registerSocketEvent()
})
// #endregion

// #region browser event handler
// 投稿メッセージをサーバに送信する
const onPublish = () => {
  if(!chatContent.value) {
    return
  }
  if (selectedDate.value) {
    if(selectedDate.value < jstDatetime) {
      alert("現在時刻以降を選択してください")
      return
    }
  }
  console.log(selectedDate.value)
  // @担当者名を付けた投稿メッセージを作成
  const messageWithTo = toWho.value ? `@${toWho.value} ${chatContent.value}` : chatContent.value
  socket.emit("publishEvent", `${userName.value}さん: ${messageWithTo}`)
  //いつの誰に向けてのメッセージかを追加
  if (toWho.value || selectedDate.value) {
    socket.emit("publishTask", {
      who: toWho.value,
      when: selectedDate.value ? selectedDate.value.replace('T', ' ').replace(/-/g, '/') : selectedDate.value.replace("*",""),
      what: chatContent.value
    })
  }
  // 入力欄を初期化
  chatContent.value=""
  toWho.value=""
  selectedDate.value=""
  
}

// 退室メッセージをサーバに送信する
const onExit = () => {
  socket.emit("exitEvent", `${userName.value}さんが退出しました。`)
}

// メモを画面上に表示する
const onMemo = () => {
  // メモの内容を表示
  chatList.unshift(`${userName.value}さんのメモ: ${chatContent.value}`)
  // 入力欄を初期化
  chatContent.value=""

}
// #endregion

// #region socket event handler
// サーバから受信した入室メッセージ画面上に表示する
const onReceiveEnter = (data) => {
  chatList.unshift(data)
}

// サーバから受信した退室メッセージを受け取り画面上に表示する
const onReceiveExit = (data) => {
  chatList.unshift(data)
}

// サーバから受信した投稿メッセージを画面上に表示する
const onReceivePublish = (data) => {
  chatList.unshift(data)
}

// サーバから受信したタスクを画面上に表示する
const onReceiveTask = (data) => {
  taskList.unshift(data)
  sortByWhen()
}
// #endregion

// #region local methods
// イベント登録をまとめる
const registerSocketEvent = () => {
  // 入室イベントを受け取ったら実行
  socket.on("enterEvent", (data) => {
    onReceiveEnter(data)
  })

  // 退室イベントを受け取ったら実行
  socket.on("exitEvent", (data) => {
    onReceiveExit(data)
  })

  // 投稿イベントを受け取ったら実行
  socket.on("publishEvent", (data) => {
    onReceivePublish(data)
  })

  // タスク追加イベントを受け取ったら実行
  socket.on("publishTask", (data) => {
    onReceiveTask(data)
    sortByWhen()
  })
}

  // タスクリストをソート
  const sortByWhen = () => {
    // 期限を昇順にソート
    taskList.sort( (a, b) => {
      if(a.when === "" && b.when !== "") return 1;
      if(a.when !== "" && b.when === "") return -1;
      if(a.when === "" && b.when === "") return 0;
      if(a.when < b.when) return -1;
      if(a.when > b.when) return 1;
      return 0;
    });
  }
// #endregion
</script>

<template>
  <div class="mx-auto my-5 px-10">
    <h1 class="text-h3 font-weight-medium">タスカル</h1>
    <div class="content-all">
    <div class="content-container">
      <!-- チャット部分 -->
      <div class="chat-section">
        <p>ログインユーザ：{{ userName }}さん</p>
        <textarea variant="outlined" placeholder="投稿文を入力してください" rows="4" class="area" v-model="chatContent"></textarea>
        <div class="mt-5">
          <button class="button-normal" @click="onPublish">投稿</button>
          <button class="button-normal util-ml-8px" @click="onMemo">メモ</button>
        </div>
        <div class="mt-5">
        <input
          class="who-and-When-Input"
          v-model="toWho"
          placeholder="誰に">
        <input class="who-and-When-Input" type="datetime-local" v-model="selectedDate" :min="minDateTime">
      </div>
        <div class="mt-5" v-if="chatList.length !== 0">
          <ul>
            <li class="item mt-4" v-for="(chat, i) in chatList" :key="i">{{ chat }}</li>
          </ul>
        </div>
      </div>
    </div>

      <!-- タスクリスト部分 -->
    <div class="task-section">
      <h2 class="task-title">タスク</h2>
      <div class="task-list">
        <div class="task-header">
          <span class="task-who">担当者</span>
          <span class="task-when">期限</span>
          <span class="task-what">内容</span>
        </div>
        <div
          class="task-item"
          v-for="(task, index) in taskList"
          :key="index"
          :class="{
            overdue: task.when && new Date(task.when.replace(/\//g, '-').replace(' ', 'T')) < new Date(Date.now() - 60000),
            me: task.who === userName}"
        >
          <span class="task-who">{{ task.who }}</span>
          <span class="task-when">{{ task.when }}</span>
          <span class="task-what">{{ task.what }}</span>
        </div>
      </div>
    </div>
  </div>

    <router-link to="/" class="link">
      <button type="button" class="button-normal button-exit" @click="onExit">退室する</button>
    </router-link>
  </div>
</template>

<style scoped>
.link {
  text-decoration: none;
  position: absolute;
  top: 0;
  right: 0;
}

.content-all {
  display: flex;
  flex-direction: row;
  gap: 10vw;
  margin-top: 60px;
}

.content-container {
  display: flex;
  flex-direction: column;
  gap: 40px;
  margin-top: 20px;
  flex: 1;
  max-width: 50%;
}

.chat-section {
  flex: 1;
}

.task-section {
  flex: 1;
  min-width: 400px;
}

.task-title {
  font-size: 1.5rem;
  font-weight: bold;
  margin-bottom: 16px;
  color: #333;
}

.task-list {
  border: 1px solid #ddd;
  border-radius: 4px;
  overflow: hidden;
}

.task-header {
  display: flex;
  background-color: #f5f5f5;
  font-weight: bold;
  border-bottom: 2px solid #ddd;
}

.task-item {
  display: flex;
  border-bottom: 1px solid #eee;
}

.task-item:last-child {
  border-bottom: none;
}

.task-item:hover {
  background-color: #f9f9f9;
}

.task-who,
.task-when,
.task-what {
  padding: 12px 16px;
  flex: 1;
  border-right: 1px solid #eee;
}

.task-who,
.task-when,
.task-what:last-child {
  border-right: none;
}

.task-who {
  min-width: 100px;
}

.task-when {
  min-width: 140px;
}

.task-what {
  min-width: 160px;
}

.area {
  width: 100%;
  max-width: 500px;
  border: 1px solid #000;
  margin-top: 8px;
}

.item {
  display: block;
}

.util-ml-8px {
  margin-left: 8px;
}

.button-exit {
  color: #000;
  margin-top: 8px;
}

.who-and-When-Input {
  border: 1px solid #000;
  border-radius: 5px;
  padding: 3px;
}

.task-item.overdue {
  color: #f44336;
}

.task-item.overdue:hover {
  color: #d32f2f;
}

.task-item.me {
  background-color: #7ff436;
}

.task-item.me:hover {
  background-color: #5ab521;
}
</style>