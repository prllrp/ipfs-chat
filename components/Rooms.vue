<template>
    <div class="main">
       <!-- List the chat rooms that are announced to us and show their info -->
        <div class="rooms">
            <div class="room" v-for="room in rooms" :key="room.id">
                <div class="room-info">
                    <div class="room-name">{{ room.name }}</div>
                    <div class="room-description">{{ room.description }}</div>
                </div>
                <div class="room-join">
                    <button @click="joinRoom(room.id)">Join</button>
                </div>
            </div>
        </div>
        <!-- A form for creating a new room -->
        <div class="create-room">
            <div class="create-room-name">
                <label for="create-room-name">Room Name</label>
                <input type="text" id="create-room-name" v-model="newRoomName" />
            </div>
            <div class="create-room-description">
                <label for="create-room-description">Room Description</label>
                <input type="text" id="create-room-description" v-model="newRoomDescription" />
            </div>
            <div class="create-room-submit">
                <button @click="createRoom()">Create Room</button>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: "Rooms",
    data() {
        return {
            //rooms: [],
            newRoomName: "",
            newRoomDescription: ""
        };
    },
    props: {
        rooms: {
            type: Array,
            default: []
        }
    },
    methods: {
        async createRoom() {
            // Create a new room
            const room = {
                name: this.newRoomName,
                description: this.newRoomDescription
            };
            // Announce the room to the network
            this.$emit("createRoom", room);
            // Reset the form
            this.newRoomName = "";
            this.newRoomDescription = "";
        },
        async joinRoom(roomId) {
            // Join the room
            this.$emit('joinChannel', roomId);
            
        }
    },
    created() {
        
    }
};


</script>

<style>

</style>