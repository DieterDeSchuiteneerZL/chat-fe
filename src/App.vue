<template>
  <el-container >
    <el-main :style="{maxWidth: '420px'}">
      <el-row>
        <el-col :span="24">
          <el-scrollbar height="420px" class="border">
            <template v-for="({message, name, type}, index) in messages" :key="`message_${index}`">
              <i v-if="type==='action'" class="info">{{message}}</i>     
              <p v-else  v-html="`${name ?? 'Anonymmous'}: ${message}`"/>         
            </template>
          </el-scrollbar>
        </el-col>
      </el-row>
      <el-row justify="space-between" :style="{marginTop: '10px'}">
        <el-col :span="19">
          <el-input v-model="message" placeholder="your message to better the world" v-on:keypress.enter="sendMessage"/>
        </el-col>
        <el-col :span="4">
          <el-button round type="primary" @click="sendMessage">Send</el-button>
        </el-col>
      </el-row>
    </el-main>
  </el-container>
</template>

<script setup>
  import { io } from "socket.io-client";
  import { ElMessage } from 'element-plus'

  import { ref, onMounted } from 'vue'
  const WEBSOCKET = "https://4k2zqg.sse.codesandbox.io/" 
  
  const socket = io(WEBSOCKET, { reconnectionDelayMax: 10000 })
  
  const message = ref('')
  const messages = ref([])

  socket.on("connect", () => {
    socket.connected ? ElMessage({message: "Connected to the chat", type: 'success'}) : ElMessage({message: "Connected to the chat", type: 'error'})
    console.log(socket.connected? "connected" : "error connecting to socket"); // true
  });

  socket.on("message", (message) => {
    messages.value = [...messages.value, message]
  });

  socket.on("action", (action) => {
    messages.value = [...messages.value, {...action, type: 'action'}]
  });

  function sendMessage () {
    console.log("send message", message.value)  
    socket.emit("message", message.value)
    message.value = ''
  }

  onMounted(() => {
    socket.emit("userConnect")
    socket.emit("login", "Sal Ami")
  })
</script>

<style scoped>
.info {
    color: var(--el-color-info);

}
p, i  {
  font-family: Arial;
  display: flex;
  align-items: center;
  justify-content: left;
  height: 24;
  margin: 5px;
  color: var(--el-color-lighterill);
}
.border {
  border: 1px solid var(--el-color-primary);
  border-radius: 5px;
}
</style>