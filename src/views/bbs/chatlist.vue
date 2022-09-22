<!-- eslint-disable vue/no-deprecated-filter -->
<!-- eslint-disable vue/require-v-for-key -->
<!-- 聊天列表 -->
<template>
<div class="msglist">
    <ul>
        <li v-for="item in channel" class="sessionlist" :class="{ active: item.cid === selectId }" @click="select(item.cid)" :key="item.cid">
            <div class="list-left">
                <img class="avatar" width="42" height="42" :alt="item.name" :src="item.picSrc">
            </div>
            <div class="list-right">
                <p class="name">{{item.name}}</p>
                <span class="time">{{TimeCurTime(new Date(lastDate[item.cid]))}}</span>
                <p class="lastmsg">{{lastMessage[item.cid]}}</p>
            </div>
        </li>
    </ul>
</div>
</template>

<script lang="ts">
import { HttpManager } from "@/api";
import {computed, ref, watch} from "vue";
import {
    useStore,
    mapState,
    mapActions,
    mapGetters
} from 'vuex'
import { get } from '@/api/request';
export default {
    setup() {
        const {state} = useStore();
        const channel = ref([]);
        const tempChannel = ref([]);
        async function getData() {
            const result = (await HttpManager.getChannels()) as ResponseBody;
            channel.value = result.data;
            tempChannel.value = result.data;
        }
        getData();
        function select(id) {
            state.selectId = id;
        }
        let id = computed(() => {return state.selectId});
        let searchWord = computed(() => {return state.searchText});
        watch(id, () => {
            const item = channel.value.find(item => item.cid == state.selectId);
            // console.log(channel.value);
            state.channelname = item.name;
            state.channelImg = item.picSrc;
        });
        watch(searchWord, () => {
            if (searchWord.value === "") {
                channel.value = tempChannel.value;
            } else {
                channel.value = [];
                for (let item of tempChannel.value) {
                if (item.name.includes(searchWord.value)) {
                    channel.value.push(item);
                }
                }
            }
        });
        return {
            channel,
            state,
            select,
            getData
        };
    },
    computed: {
        ...mapState([
            'selectId',
            'lastMessage',
            'searchText',
            'lastDate'
        ]),
        ...mapGetters([
            'searchedChatlist'
        ]),
        TimeCurTime(date) {
            return function(date) {
                if (typeof date === 'string' || date == null) {
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
    methods: {
        ...mapActions([
            'selectSession',
        ]),
        lastMsg(id){
            const {state} = useStore();
            console.log(state.lastMessages[id].content);
            return state.lastMessages[id]["content"];
        }
    }
}
</script>

<style lang="stylus" scoped>
.msglist
  height: 540px
  overflow-y: auto
  .sessionlist
    width: 360px
    display: flex
    padding: 12px
    transition: background-color .1s
    font-size: 0
    &:hover 
        background-color: rgb(220,220,220)
    &.active 
        background-color: #c4c4c4
    .avatar
        border-radius: 2px
        margin-right: 12px
    .list-right
        position: relative
        flex: 1
        margin-top: 4px
        .name
            display: inline-block
            vertical-align: top
            font-size: 14px
        .time
            float: right
            color: #999
            font-size: 10px
            vertical-align: top
        .lastmsg
            position: absolute
            font-size: 12px
            width: 130px
            height: 15px
            line-height: 15px
            color: #999
            bottom: 4px
            overflow: hidden
            white-space:nowrap
            text-overflow:ellipsis
</style>

<style scoped>
html, body, div, span, applet, object, iframe,
h1, h2, h3, h4, h5, h6, p, blockquote, pre,
a, abbr, acronym, address, big, cite, code,
del, dfn, em, img, ins, kbd, q, s, samp,
small, strike, strong, sub, sup, tt, var,
b, u, i, center,
dl, dt, dd, ol, ul, li,
fieldset, form, label, legend,
table, caption, tbody, tfoot, thead, tr, th, td,
article, aside, canvas, details, embed,
figure, figcaption, footer, header,
menu, nav, output, ruby, section, summary,
time, mark, audio, video, input {
    margin: 0;
    padding: 0;
    border: 0;
    font-size: 100%;
    font-weight: normal;
    vertical-align: baseline;
}

article, aside, details, figcaption, figure,
footer, header, menu, nav, section {
    display: block;
}

body {
    line-height: 1;
}

blockquote, q {
    quotes: none;
}

blockquote:before, blockquote:after,
q:before, q:after {
    content: none;
}

table {
    border-collapse: collapse;
    border-spacing: 0;
}

a {
    color: #7e8c8d;
    text-decoration: none;
    -webkit-backface-visibility: hidden;
}

li {
    list-style: none;
}


html, body {
    width: 100%;
    height: 100%;
    overflow-y: hidden  
}
body {
    -webkit-text-size-adjust: none;
    -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
    background-size: cover;
}
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
