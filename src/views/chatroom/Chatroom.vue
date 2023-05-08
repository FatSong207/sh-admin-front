<template>
    <div>
        <a-input v-model:value="data.msg" />
        <a-button @click="send()">send</a-button>
        <a-button @click="disConnect()">disconnected</a-button>
        <a-row :gutter="20">
            <a-col :span="18">
                <a-card class="hcard">
                    <div v-for="(item, index) in data.msgs" :key="index"
                        :class="item.from !== 'sys' ? 'chatlistdiv' : 'chatlistdiv broadcast'">
                        <span v-if="item.from !== 'sys'">
                            <!-- <a-avatar :size="18">{{ item.from.slice(0, 1) }}</a-avatar> -->
                            <span>
                                <a-avatar :size="22">{{ item.from.slice(0, 1) }}</a-avatar>
                                {{ item.from }} :
                                {{ item.content }}
                                {{ moment(item.time).format('YYYY-MM-DD HH:mm:ss') }}
                            </span>
                        </span>
                        <span v-else class="sys_content">
                            {{ item.content }}
                        </span>
                    </div>
                </a-card>
            </a-col>
            <a-col>
                <a-card title="在線用戶">
                    <div v-for="(item, index) in data.user_list" :key="index">
                        {{ item }}
                    </div>
                </a-card>
            </a-col>
        </a-row>
    </div>
</template>

<script setup>
import { reactive, onMounted, onUnmounted } from 'vue';
import { useUserStore } from '../../store/user';
import moment from 'moment'
const userStore = useUserStore()

const data = reactive({
    msgs: [],
    user_list: [],
    msg: '',
})

onMounted(() => {
    InitSocket()
})

onUnmounted(() => {
    disConnect()
})

let socket = null

const InitSocket = () => {
    // if (socket) {
    //     ElMessage.error('請勿重複登錄！')
    //     return
    // }
    socket = new WebSocket(`ws://localhost:5001/ws/${userStore.userInfo.Name}`)
    socket.onopen = () => {
        console.log('connected!', socket)
        // const sendPlayload = {
        //     type: 1,
        //     from: "from",
        //     content: "content"
        // }
        // socket.send(JSON.stringify(sendPlayload))
        // status.value = '已連線'
        // disConnect.value = false
    }

    socket.onerror = () => {
        console.log('Error!')
    }

    socket.onclose = () => {
        console.log('Onclose!', socket)
        // status.value = '已斷線'
    }

    socket.onmessage = (msg) => {
        const resMsg = JSON.parse(msg.data)
        console.log(resMsg)

        if (resMsg.type === 0) {
            data.msgs.push(resMsg)
        } else if (resMsg.type === 1) {
            console.log('type==1', resMsg)
            data.user_list = resMsg.content.split(',')
            console.log(data.user_list)
        }
        // switch (resMsg.action) {
        //     case 'Init':
        //         userlist.users = resMsg.connected_users
        //         msgArr.msgs.push(resMsg)
        //         break
        //     case 'SendMsg':
        //         console.log(resMsg)
        //         msgArr.msgs.push(resMsg)
        //         break
        //     case 'Left':
        //         userlist.users = resMsg.connected_users
        //         msgArr.msgs.push(resMsg)
        //         console.log('close!', socket)
        //         break
        //     default:
        //         break
        // }
    }
}

const disConnect = () => {
    const sendPlayload = {
        type: 2,
        from: userStore.userInfo.Name,
    }
    socket.send(JSON.stringify(sendPlayload))
}

const send = () => {
    const sendPlayload = {
        type: 0,
        from: userStore.userInfo.Name,
        content: data.msg
    }
    console.log(data.msg)
    socket.send(JSON.stringify(sendPlayload))
}

</script>

<style scoped>
.chatlistdiv {
    margin: 8px;
    border: 1px solid greenyellow;
}

.broadcast {
    text-align: center;
}

.sys_content {
    font-size: x-small;
    background-color: rgb(145, 143, 143);
    padding: 4px;
    border-radius: 6px;
    color: white;
}
</style>