<template>
  <div class="main">
    <div class="chat">
        <div class="chat-header">
            <div class="chat-header-title">
                <h3>{{ title }}</h3>
            </div>
            
        </div>
        <div class="chat-body">
            <div class="chat-body-messages">
                <div v-for="message in messages" v-bind:key="message.messageId"  class="chat-body-message">
                    <div class="chat-body-message-text">
                        {{ message.text }}
                    </div>
                    <div class="chat-body-id">
                        {{ message.from }}
                    </div>
                    <div class="chat-body-message-time">
                        {{ message.time }}
                    </div>
                </div>
            </div>
        </div>
        <div class="chat-footer">
            <div class="chat-footer-input">
                <input type="text" v-model="message" @keyup.enter="sendMessage" />
            </div>
            <div class="chat-header-buttons">
                <button class="btn btn-primary"  @click="sendMessage">Send</button>
            </div>
        </div>
    </div>
  </div>
</template>

<script>
export default {
    data(){
        return{
            message: '',
        }
    },
    methods:{
        sendMessage(){
            if(this.channelJoined){
                this.$emit('join-channel', 'global');
                console.log('No channel joined');
                return
            }
            this.$emit('send-message', this.message);
            this.message = '';
        }
    },
    props:{
        title: {
            type: String,
            default: 'Chat'
        },
        messages: {
            type: Array,
            default: () => []
        },
        channelJoined: {
            type: Boolean,
            default: false
        }
    },
    created(){
        
    },
    onMounted(){
        
    }

}
</script>

<style>
    .main{
        width: 100%;
        height: 100%;
        margin: 0px;
        padding: 0px;
    }

    .chat{
        width: 20vw;
        height: 50vh;
        display: flex;
        margin: 20px;
        padding: 20px;
        flex-direction: column;
        justify-content: center;
        align-items: center;
    }
    
    .chat-body{
        width: 100%;
        height: 100%;
        display: flex;
        flex-direction: column;
        justify-content: flex-start;
        align-items: flex-start;
        margin: 10px;
        padding: 10px;
    }

    .chat-body-message{
        width: 100%;
        height: 5vh;
        display: flex;
        flex-direction: row;
        justify-content: flex-start;
        align-items: flex-start;
    }

    .chat-body-message-id{
        width: 100%;
        height: 100%;
        display: flex;
        flex-direction: column;
        justify-content: flex-start;
        align-items: flex-start;
        margin: 10px;
        padding: 10px;
    }
</style>

<!-- Build a chat box that emits outgoing messages and displays incoming ones -->