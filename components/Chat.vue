<template>
  <div class="main">
    <div class="chat">
        <div class="chat-header">
            <div class="chat-header-title">
                <h3>{{ title }}</h3>
            </div>
            <div class="chat-header-buttons">
                <button class="btn btn-primary"  @click="sendMessage">Send</button>
            </div>
        </div>
        <div class="chat-body">
            <div class="chat-body-messages">
                <div v-for="message in messages" v-bind:key="message.messageId"  class="chat-body-message">
                    <div class="chat-body-message-text">
                        {{ message.text }}
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

</style>

<!-- Build a chat box that emits outgoing messages and displays incoming ones -->