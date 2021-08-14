<template>
    <div class="row">
        <div class="col-md-10" >
            <div class="card card-default">
                <div class="card-header">
                    <h5>Chat App</h5>
                </div>
                <div class="card-body p-0" style="height:500px;overflow-y:scroll;" v-chat-scroll>
                    <ul class="list-unstyled">
                        <li class="p-2" v-for="(message, index) in messages" :key="index" >
                            <strong>{{ message.user.name }}</strong><br>
                                {{ message.message }}
                        </li>
                    </ul>
                </div>
                <div class="card-footer">
                    <input 
                        @keyup.enter = "sendMessage"
                        v-model="newMessage" 
                        type="text" name="message" 
                        placeholder="Type a Message" 
                        class="form-control"
                    >
                </div>
            </div>
        </div>
        <div class="col-md-2">
            <div class="card card-default">
                <div class="card-header ">
                    <h5 >Active Users</h5>
                </div>
                <div class="card-body p-0" style="height:515px;">
                    <ul>
                        <li class="p-2" v-for="(user, index) in users" :key="index">
                            {{ user.name }}
                        </li>
                    </ul>
                </div>
                <div class="card-footer">
                    <span class="text-muted"></span>
                </div>
            </div>
        </div>
    </div>
</template>

<script>

    export default {
        props: ['user'],

        data(){
            return {
                messages:[],
                newMessage:'',
                users:[]
            }
        }, 

        created() {
            this.fetchMessages();

            Echo.join('chat')
                .here(user => {
                    this.users = user;
                })
                .joining(user => {
                    this.users.push(user);
                })
                .leaving(user => {
                    this.users = this.users.filter(u => u.id != user.id);
                })
                .listen('MessageSent',(event) => {
                    this.messages.push(event.message);
            });
        },

        methods: {
                fetchMessages() {
                    axios.get('messages').then(response => {
                        this.messages = response.data;
                    })
                },
                sendMessage() {

                    this.messages.push({
                        user: this.user,
                        message: this.newMessage
                    });

                    axios.post('messages',{message: this.newMessage});

                    this.newMessage = '';
                }
        }
    }
</script>
