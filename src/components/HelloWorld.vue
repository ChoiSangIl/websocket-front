<template>
  <div>
    <button @click="send">click me</button>
  </div>
</template>

<script>
import Stomp from 'webstomp-client'
import SockJS from 'sockjs-client'

export default {
  name: 'App',
  data() {
    return {
    }
  },
  created() {
    const serverURL = "http://localhost:8080/sit-down-monkey"
    let socket = new SockJS(serverURL);
    this.stompClient = Stomp.over(socket);
    
    console.log(`소켓 연결을 시도합니다. 서버 주소: ${serverURL}`)
    this.stompClient.connect(
        {},
        frame => {
          this.connected = true;
          console.log('소켓 연결 성공', frame);
            this.stompClient.subscribe("/topic", res => {
                console.log('구독으로 받은 메시지 입니다.', res.body);
            });
        },
        error => {
          console.log('소켓 연결 실패', error);
          this.connected = false;
        } 
    );        
  },
  methods:{
    send() {
        if (this.stompClient && this.stompClient.connected) {
            this.stompClient.send("/send", JSON.stringify("test"), {});
        }
    }
  }
}
</script>