<template>
  <div id="list">
  	<ul v-if="currentList=='群聊'">
<!--群聊列表-->
			<p style="padding: 2px 4px;height: 20px">群聊列表</p>
			<li :class="{ active: currentSession?'群聊'== currentSession.username:false }"
					v-on:click="changeCurrentSession(chatObj)">
				<img class="avatar" src="https://ss1.bdstatic.com/70cFuXSh_Q1YnxGkpoWK1HF6hhy/it/u=1268761962,3976237305&fm=26&gp=0.jpg">
				<el-badge :is-dot="isDot[user.username+'#群聊']"><p class="name">群聊</p></el-badge>
			</li>
			</ul>
<!--机器人-->
		<ul v-if="currentList=='机器人'">
			<p style="padding: 2px 4px;height: 20px">快来和机器人聊天吧！</p>
			<li :class="{ active: currentSession?'机器人'== currentSession.username:false }"
					v-on:click="changeCurrentSession(robotObj)">
				<img class="avatar" src="https://ss0.bdstatic.com/70cFuHSh_Q1YnxGkpoWK1HF6hhy/it/u=2548892998,499717296&fm=26&gp=0.jpg">
				<p class="name">瓦力(智能回复)</p>
			</li>
		</ul>
<!--用户列表-->
		<el-scrollbar      wrap-class="userList" wrap-style="height:600px;"
											  view-style="height:100%;" :native="false">
		<ul v-if="currentList=='私聊'" >
			<p style="padding: 2px 4px;height: 20px">用户列表</p>
  		<li v-for="item in users" :class="{ active: currentSession?item.username === currentSession.username:false }"
					v-on:click="changeCurrentSession(item)"><!--   :class="[item.id === currentSession ? 'active':'']" -->
				<div style="display: flex;justify-content: space-between">
					<div>
						<el-badge :is-dot="isDot[user.username+'#'+item.username]" style="">
							<el-image class="avatar"
												:preview-src-list="[item.userProfile]"
												:src="item.userProfile"
												:alt="item.nickname">
								<div slot="error" class="image-slot">
									<i class="el-icon-picture-outline"></i>
								</div>
							</el-image>
						</el-badge>
						<p class="name">{{item.nickname}}</p>
					</div>
					<div>
					<el-badge :value="item.userStateId==1?'在线':'离线'" :type="item.userStateId==1?'danger':'info'"></el-badge>
					</div>
				</div>
  		</li>
  	</ul>
		</el-scrollbar>
  </div>
</template>

<script>
import {mapState} from 'vuex'

export default {
  name: 'list',
  data () {
    return {
			user:this.$store.state.currentUser,
			chatObj:{username:'群聊',nickname:'群聊'},//群聊实体对象（为方法复用而构造，对于User对象）
			robotObj:{
				username:'机器人',
				nickname:'机器人',
			  userProfile:'https://ss0.bdstatic.com/70cFuHSh_Q1YnxGkpoWK1HF6hhy/it/u=2548892998,499717296&fm=26&gp=0.jpg'
			}
    }
  },
  computed: mapState([
  //'sessions',//this.sessions映射成this.$store.state.sessions
	'users',
  'currentSession',
	'isDot',
	'currentList'
	]),
  methods:{
	changeCurrentSession(currentSession) {
    this.$store.commit('changeCurrentSession', currentSession);
    // 如果是私聊且不是群聊或机器人，尝试检查或创建房间
    if (currentSession.username !== '群聊' && currentSession.username !== '机器人') {
      this.initPrivateChat(currentSession);
    }
  },
  initPrivateChat(currentSession) {
      // 构建请求的payload，通常JSON的属性名使用小写
      const payload = {
        username: this.user.username, // 当前登录用户的用户名
        otherUsername: currentSession.username // 选择进行私聊的用户的用户名
      };
    
      // 发送POST请求
      this.postRequest("/userchat/conversations/join", payload)
        .then(response => {
          this.$store.state.conversation = response.conversation;
        //console.log("11Room ID:", response);
          // 检查后端是否返回了预期的响应

        })
        .catch(error => {
          // 错误处理
          console.error("Error initializing private chat:", error);
          if (error.response) {
            // 服务器响应了请求，但状态码不是2xx
            console.log("Error response data:", error.response.data);
            console.log("Error response status:", error.response.status);
          } else if (error.request) {
            // 请求已发出，但没有收到响应
            console.log("No response received:", error.request);
          } else {
            // 在设置请求时发生了某些事情
            console.log("Error setting up request:", error.message);
          }
        });
		this.getRequest("/userchat/conversations/41/messages").then(resp=>{
          if (resp){
			console.log("11112111Error response data:", resp);
            //Vue.set(state.sessions,'群聊',resp);
          }
        })
    }


  }
}
</script>

<style lang="scss" scoped>
#list {
	ul{
		margin-left: 0px;
		padding-left: 0px;
		margin-left: 2px;
	}
	li {
		padding-top: 14px;
		padding-bottom: 14px;
		//padding-right: 40px;
		//border-bottom: 1px solid #292C33;
		list-style: none;
		cursor: pointer;
		&:hover {
			background-color: #D8D6D6;
		}
	}
  li.active {/*注意这个是.不是冒号:*/
			background-color: #C8C6C6;
	}
	.avatar {
		border-radius: 2px;
		width: 30px;
		height: 30px;
		vertical-align: middle;
	}
	.name {
		display: inline-block;
		margin-left: 15px;
		margin-top: 0px;
		margin-bottom: 0px;
	}
	.stateItem {//在线状态的样式
		/*position: absolute;*/
		/*left: 160px;*/
		//margin-left: 100px;
		//margin-right: 10px;
	}
	.userList{
		max-height: 600px;
	}
	.el-scrollbar__wrap.default-scrollbar__wrap {
		overflow-x: auto;
	}
}
</style>
