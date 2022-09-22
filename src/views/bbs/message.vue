<!-- eslint-disable vue/no-deprecated-filter -->
<!-- 消息框 -->
<template>
<div class="message">
    <header class="header" align="center">
        <div class="friendname">{{channelname}}</div>
    </header>
    <div class="message-wrapper" ref="list" id="messageList">
        <ul>
            <li v-for="(item,i) in messages" class="message-item" :key="i">
                <div class="time"><span>{{TimeCurTime(new Date(item.date))}}</span></div>
                <div class="main" :class="{ self: item.uid == userId }">
                    <img class="avatar" width="36" height="36" :src="rePic(item.uid)" />
                    <div class="content">
                        <div class="text" v-html="replaceFace(item.content)"></div>
                    </div>
                </div>
            </li>
        </ul>
    </div>
</div>
</template>

<script lang="ts">
import { HttpManager } from "@/api";
import { get } from "@/api/request";
import {defineComponent, ref, watch, computed} from "vue";
import {
    useStore,
    mapGetters,
    mapState
} from 'vuex'
export default defineComponent({
    setup() {
        const {state} = useStore();
        const messages = ref([]);
        async function getData(id) {
            const result = (await HttpManager.getMessage(id)) as ResponseBody;
            messages.value = result.data;
            state.lastMessage[id] = result.data[result.data.length - 1].content;
            state.lastDate[id] = result.data[result.data.length - 1].date;
            state.users = (await HttpManager.getUsers() as ResponseBody).data;
        }
        function rePic(id) {
            for (var i = 0; i < state.users.length; i++) {
                if (state.users[i].uid == id) return state.users[i].picSrc;
            }
             return "";
        }
        //使用计算属性动态拿到vuex的值
        let id = computed(() => {return state.selectId});
        let send = computed(() => {return state.isSend});
        getData(id.value);
        watch(id, () => {
            getData(id.value);
        },{immediate:true,deep: true});
        watch(send, () => {
            getData(id.value);
        },{immediate:true,deep: true});
        return {
            messages,
            rePic,
            getData,
            state,
            id,
            send
        };
    },
    computed: {
        ...mapGetters([
            'selectedChat',
            // 'messages',
        ]),
        ...mapState([
            'user',
            'users',
            'userId',
            'emojis',
            'selectId',
            'channelname',
            'channelImg',
            'lastMessage'
        ]),
        TimeCurTime(date) {
            return function(date) {
                if (typeof date === 'string') {
                    date = new Date(date);
                }
                if(date.getMinutes()<10){
                  return date.getHours() + ':0' +date.getMinutes();
                }else{
                  return date.getHours() + ':' + date.getMinutes();
                }
            }
        }
    },
    mounted() {
        //  在页面加载时让信息滚动到最下面
        var messageList = document.getElementById("messageList");
        setTimeout(() => messageList.scrollTop = messageList.scrollHeight, 0);
    },
    watch: {
        // 发送信息后,让信息滚动到最下面
        messages() {
            var messageList = document.getElementById("messageList");
            setTimeout(() => messageList.scrollTop = messageList.scrollHeight, 0);
        }
    },
    methods: {
        //  在发送信息之后，将输入的内容中属于表情的部分替换成emoji图片标签
        //  再经过v-html 渲染成真正的图片
        replaceFace(con) {
            if (con.includes('/:')) {
                var emojis = this.emojis;
                for (var i = 0; i < emojis.length; i++) {
                    con = con.replace(emojis[i].reg, '<img src="static/emoji/' + emojis[i].file + '"  alt="" style="vertical-align: middle; width: 24px; height: 24px" />');
                }
                return con;
            }
            return con;
        }
    }
})
</script>

<style lang="stylus" scoped>
   .message
      width: 100%
      height: 450px
      .header
        height: 60px
        padding: 28px 0 0 30px
        box-sizing: border-box
        border-bottom: 1px solid #e7e7e7
        .friendname
            font-size: 18px
      .message-wrapper
        min-height: 390px
        max-height: 390px
        padding: 10px 15px
        box-sizing: border-box
        overflow-y: auto
        border-bottom: 1px solid #e7e7e7
        .message
            margin-bottom: 15px
        .time
            width: 100%
            font-size: 12px
            margin: 7px auto
            text-align: center
            span
                display: inline-block
                padding: 4px 6px
                color: #fff
                border-radius: 3px
                background-color: #dcdcdc
        .main
            .avatar 
                float: left
                margin-left: 15px
                border-radius: 3px
            .content
                display: inline-block
                margin-left: 10px
                position: relative
                padding: 6px 10px
                max-width: 330px
                min-height: 36px
                line-height: 24px
                box-sizing: border-box
                font-size: 14px
                text-align: left
                word-break: break-all
                background-color: #fafafa
                border-radius: 4px
                &:before
                    content: " "
                    position: absolute
                    top: 12px
                    right: 100%
                    border: 6px solid transparent
                    border-right-color: #fafafa
        .self
            text-align: right
            .avatar
                float: right
                margin:0 15px
            .content 
                background-color: #b2e281
                &:before 
                    right: -12px
                    vertical-align: middle
                    border-right-color: transparent
                    border-left-color: #b2e281
</style>

<style scoped>
    /* 设置滚动条的样式 */
::-webkit-scrollbar {
    width: 8px;
}
/* 滚动条滑块 */
::-webkit-scrollbar-thumb {
    border-radius: 6px;
    background: rgba(0,0,0,0.1);
}
</style>