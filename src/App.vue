<template>
  <div id="app">
    <button v-on:click="onTest">{{uid}}</button>
    <iframe
      ref="chat"
      :src="imURL"
      style="width:100%;height:100vh"
    />
  </div>
</template>

<script>
import lim from "limao";
export default {
  name: "App",
  data() {
    const uid = "555539517403037720"
    const token ="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOlsib2F1dGgyLXJlc291cmNlIl0sInVzZXJfbmFtZSI6ImhlaG9uZ2JpbmciLCJzY29wZSI6WyJhbGwiXSwiZXhwIjoxNTg1MzAwMjIxLCJhdXRob3JpdGllcyI6WyJST0xFX1VTRVIiXSwianRpIjoiNjBjZWUyNmMtZTBkNy00ODgyLTliNTItNmIxODIyYmY5NGUxIiwiY2xpZW50X2lkIjoiY29jbyJ9.xhPaaqqb3rFeLIJoUXCacVDm9OhCG4Dsy-irvCS1mpw"
     return {
       uid: uid,
       token: token,
       imURL: `http://localhost:3000?uid=${uid}&token=${token}&channel_id=636595900868722688&channel_type=1&apiURL=https://appuat.cocospace.com.cn/api/`
     }
  },
  mounted() {
    lim.options.connectURL = "ws://116.62.53.36:8030/"
    // 监听连接状态
    lim.addConnectStatusListener((status,reasonCode)=>{
      console.log('status->',status,reasonCode);
      if(status == 1) {
        console.log('连接成功！');
      }else {
        console.log('连接失败！');
      }
    });
    // IM消息状态改变
    lim.addMessageStatusListener((sendackPacket)=>{
         const limWindow = this.$refs.chat.contentWindow;
         let newSendackPacket = {...sendackPacket}
         newSendackPacket.messageID = sendackPacket.messageID.toString() // 消息编号是bignumber类型 需要特殊处理下
         limWindow.postMessage({type:"sendack",data:newSendackPacket},"*");
    });
     // 监听消息
    lim.addMessageListener((message)=>{
        console.log('收到消息->',message);
        let newMessage = {...message}
        const limWindow = this.$refs.chat.contentWindow;
        newMessage.messageID = message.messageID.toString();
        limWindow.postMessage({type:"recv",data:newMessage},"*");
    });

    // 监听子窗口发过来的请求
    window.addEventListener("message",(event)=>{
       if(!event.data) {
         return;
       }
       console.log('event->',event);
       let type = event.data.type;
       if(type === "send") { // 发送消息
          lim.sendMessage(event.data.payload,event.data.channel);
       }
    }) 
    // 连接IM
    console.log('this.uid--->',this.uid);
    console.log('this.token--->',this.token);
    lim.connect(this.uid,this.token);

  },
  components: {},
  methods: {
    onTest() {
    // this.$refs.chat.contentWindow.lim11 = "dsdsdszzz";
    console.log('this.$refs.chat.contentWindow->',this.$refs.chat.contentWindow);
    }
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin: 0 auto;
  width: 100%;
  height: 100%;
}
</style>
