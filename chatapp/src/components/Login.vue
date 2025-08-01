<script setup>
import { inject, ref } from "vue"
import { useRouter } from "vue-router"
import socketManager from '../socketManager.js'

// #region global state
const userName = inject("userName")
// #endregion

// #region local variable
const router = useRouter()
const socket = socketManager.getInstance()
// #endregion

// #region reactive variable
const inputUserName = ref("")
// #endregion

// #region browser event handler
// 入室メッセージをクライアントに送信する
const onEnter = () => {
  console.log("Enterが押されました")
  // ユーザー名が入力されているかチェック
  if (inputUserName.value.trim().length !== 0) { // ユーザ名が入力されている場合
    // 入室メッセージを送信
    const message = `${inputUserName.value}さんが入室しました。`;
    socket.emit("enterEvent", message);
    // 全体で使用するnameに入力されたユーザー名を格納
    userName.value = inputUserName.value;
    // チャット画面へ遷移
    router.push({ name: "chat" });
  } else { // ユーザ名が入力されていない場合
    alert("名前を入力してください。");
  }
}
// #endregion
</script>

<template>
  <div class="d-flex flex-column justify-center align-center h-screen bg-grey-lighten-3">
    <v-card
      class="pa-12 d-flex flex-column align-center"
      width="400"
      elevation="8"
    >
      <h1 class="text-h4 font-weight-bold mb-6">タスカル</h1>
      <v-form @submit.prevent="onEnter">
        <v-text-field
          clearable
          label="名前"
          variant="outlined"
          class="user-name-text mb-4"
          v-model="inputUserName"
          style="width: 250px"
        />
        <v-btn type="submit" block class="button-normal" color="primary">入室する</v-btn>
      </v-form>
    </v-card>
    
    <v-footer
      class="text-center py-4"
      color="blue-grey-darken-3"
      dark
      style="position: fixed; bottom: 0; width: 100%;"
    >
      <v-container>
        <p class="text-caption">© 2025 Team-o-f</p>
      </v-container>
    </v-footer>
  </div>
</template>

<style scoped>
</style>
