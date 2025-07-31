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
  <div style="text-align: center;max-height: 100vh;" class="pt-10">
    <h1 class="text-h3 font-weight-medium">タスカル</h1>
    <v-form @submit.prevent="onEnter">
      <div class="mt-10">
        <v-text-field clearable label="名前" variant="outlined" class="user-name-text" v-model="inputUserName" style="width: 200px; display: inline-block; padding: 10px;"/>
      </div>
        <v-btn type="submit" @click="onEnter"  class="button-normal">入室する</v-btn>
    </v-form>
    
    <v-footer
    class="text-center py-4"
    color="blue-grey-darken-3"
    dark
    style="position: absolute; bottom: 0; width: 100%;"
    >
      <v-container class="px-0">
        <p class="text-caption" style="margin: 0 auto;">© 2025 Team-o-f</p>
      </v-container>
    </v-footer>
  </div>
</template>

<style scoped>
</style>
