<template>
  <div>
    <template v-for="(item, index) in seatList" :key="item">
      <template v-if="index % 10 == 0"> <br/> </template>
      <v-chip v-if="item" @click="cancle(index)">
        {{ item }}
      </v-chip>

      <v-chip v-else-if="item == false" color="green" @click="reservation(index)">
        {{ item }}
      </v-chip>
    </template>
    
  </div>
</template>

<script>
import Stomp from 'webstomp-client'
import SockJS from 'sockjs-client'
import axios from 'axios';

export default {
  name: 'App',
  data() {
    return {
      seatList: Array.from({length: 100}, () => false),
      name: 'kukaro',
      age: 26,
    }
  },
  created() {
    const serverURL = "http://localhost:8080/sit-down-monkey"
    let socket = new SockJS(serverURL);
    this.stompClient = Stomp.over(socket);
    
    console.log(`소켓 연결을 시도합니다. 서버 주소: ${serverURL}`)
    let ref = this
    this.stompClient.connect(
        {},
        frame => {
          this.connected = true;
          console.log('소켓 연결 성공', frame);
            this.stompClient.subscribe("/seat", res => {
                console.log('구독으로 받은 메시지 입니다.', res.body)
                let seat = JSON.parse(res.body)
                ref.seatList[seat['id']] = seat['hasReservation']
            });
        },
        error => {
          console.log('소켓 연결 실패', error);
          this.connected = false;
        } 
    );        
  },
  methods:{
    reservation(seatId) {
      axios.get( "/api/v1/reservation/" + seatId)
        .then((response) => {
          console.log(response)
        })
        .catch((error)=>{
            console.log(error)
            alert("서버가 아파요 ㅜㅜ")
        })
    },
    cancle(seatId){
      if(confirm(seatId + "취소하시겠습니까?")){
        axios.get( "/api/v1/reservation/cancel/" + seatId)
        .then((response) => {
          console.log(response)
        })
        .catch((error)=>{
            console.log(error)
            alert("서버가 아파요 ㅜㅜ")
        })
      }
    }
  }
}
</script>