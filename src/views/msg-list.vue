<!--消息选项卡-->
<template>
  <div id="mainBody">
    <ul class="list-panel" ref="listPanel" v-if="messageList.length > 0">
      <li
        class="row-panel"
        v-for="(item, index) in messageList"
        :key="item.id"
        @click="showChatInterface(item.id, index, item.type)"
        :class="{ active: currentIndex === index }"
        v-right-click="rightMenuObj"
      >
        <div class="item-subscript-panel"></div>
        <div class="item-avatar-panel">
          <img :src="item.avatarSrc" alt="用户头像" />
        </div>
        <div class="item-nickname-panel">
          <p class="title">{{ item.userName }}</p>
          <p class="subtitle">
            {{ item.lastMsgTxt ? item.lastMsgTxt : "暂无消息" }}
          </p>
        </div>
        <div class="item-time-panel">
          <p>{{ item.lastTime }}</p>
          <!--          <div class="msg-count-panel msg-count-more-panel">-->
          <!--            <p></p>-->
          <!--          </div>-->
        </div>
      </li>
    </ul>
    <div class="content-panel" ref="contentPanel">
      <img
        src="@/assets/img/list/contact_non_selected@2x.png"
        width="280"
        height="280"
        alt="空组件"
        v-if="widgetIsNull"
      />
      <message-display
        v-else
        :message-status="messageType"
        :list-id="listId"
        @update-last-message="updateLastMessage($event)"
      />
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import _ from "lodash";
import messageDisplay from "@/components/message-display.vue";
import {
  msgListDataType,
  responseDataType,
  totalMessage
} from "@/type/ComponentDataType";

export default defineComponent({
  name: "msg-list",
  components: {
    messageDisplay
  },
  methods: {
    getThisWindowWidth: () => window.innerWidth,
    // 获取子组件传过来的值
    updateLastMessage: function(data: string) {
      this.lastMessageContent = data;
    },
    // 显示消息面板组件
    showChatInterface: function(listID: number, liIndex: number, type: number) {
      if (_.isNull(listID)) {
        alert("无消息id");
        return false;
      }
      this.currentIndex = liIndex;
      this.listId = listID;
      this.messageType = type;
      this.widgetIsNull = false;
    }
  },
  mounted() {
    // 获取好友列表
    this.$api.messageListAPI
      .getMessageList({
        pageNo: 1,
        pageSize: 20,
        userId: this.$store.state.userID
      })
      .then((res: responseDataType) => {
        if (res.code === 0) {
          this.msgList = res.data.messageList;
        } else {
          alert(res.msg);
        }
      });
  },
  data(): msgListDataType {
    return {
      msgSubscriptIco: require("../assets/img/msg-subscript.png"),
      GroupBlocked: require("../assets/img/Group_Blocked@2x.png"),
      lastMessageContent: "",
      currentIndex: -1, // 当前点击项索引
      widgetIsNull: true,
      // 右键菜单对象，菜单内容和处理事件
      rightMenuObj: {
        text: [
          "查看资料",
          "复制用户id",
          "移除该会话",
          "在联系人中查看",
          "在单聊窗口中打开",
          "会话置顶"
        ],
        handler: {
          checkingData() {
            console.log("查看资料点击事件");
          },
          copyId() {
            console.log("复制用户id点击事件");
          },
          removeItem() {
            console.log("移除会话点击事件");
          },
          showContact() {
            console.log("在联系人中查看");
          },
          showSingleChat() {
            console.log("在单聊窗口中打开");
          },
          topConversation() {
            console.log("会话置顶");
          }
        }
      },
      listId: null,
      messageType: null,
      msgList: []
    };
  },
  computed: {
    // 处理好友列表
    messageList(): Array<totalMessage> {
      const list: Array<totalMessage> = [];
      const msgList: Array<totalMessage> = this.msgList;
      for (let i = 0; i < msgList.length; i++) {
        const msgObj = msgList[i];
        // 对时间进行处理，截取小时和分钟
        msgObj.lastTime = msgObj.lastTime?.substring(10, 16);
        list.push(msgObj);
      }
      return list;
    }
  }
});
</script>

<style lang="scss" scoped src="../assets/scss/msg-list.scss"></style>
