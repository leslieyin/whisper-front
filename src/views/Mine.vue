<template>
	<div>
		<v-row>
			<v-col lg="2" class="d-none d-lg-flex"></v-col>
			<v-col lg="9">
				<v-card>
					<v-img
						src="../assets/pics/bannar_black.png"
						class="align-end"
						height="300"
						v-if="userData.bannar == undefined || userData.bannar == ''"
					>
						<v-btn class="changeBtnTop" dark @click="changeBannarWin = true"
							>更换背景图</v-btn
						>
						<v-btn class="changeBtn" @click="changeInfo">更改资料</v-btn>
					</v-img>
					<v-img :src="userData.bannar" class="align-end" height="300" v-else>
						<v-btn class="changeBtnTop" dark @click="changeBannarWin = true"
							>更换背景图</v-btn
						>
						<v-btn class="changeBtn" @click="changeInfo">更改资料</v-btn>
					</v-img>
					<v-row>
						<v-col sm="3" cols="8" offset="1" offset-sm="0">
							<v-img
								class="avatar align-center justify-center"
								src="../assets/pics/avart.png"
								v-if="userData.bannar == undefined || userData.avatar == ''"
							>
								<v-btn
									x-large
									block
									dark
									class="changAvatarBtn"
									@click="changeAvatarWin = true"
									>更换头像</v-btn
								>
							</v-img>
							<v-img
								class="avatar align-center justify-center"
								:src="userData.avatar"
								v-else
							>
								<v-btn
									x-large
									block
									dark
									class="changAvatarBtn"
									@click="changeAvatarWin = true"
									>更换头像</v-btn
								>
							</v-img>
						</v-col>
						<v-col lg="5" sm="5" cols="10" offset-sm="0">
							<v-card-title class="justify-center">{{
								userData.name
							}}</v-card-title>
							<v-card-subtitle class="justify-center d-flex">{{
								userData.intro
							}}</v-card-subtitle>
						</v-col>
						<v-col sm="4" offset-sm="0" offset="2" cols="8">
							<v-card-text class="d-flex">
								<div class="colum-line">
									发文数
									<br />
									<span class="count-text">&nbsp;{{ userData.postCount }}</span>
								</div>
								<div class="colum-line" style="border-right: none">
									评论数
									<br />&nbsp;
									<span class="count-text">{{ userData.replyCount }}</span>
								</div>
							</v-card-text>
						</v-col>
					</v-row>
				</v-card>
			</v-col>
		</v-row>

		<waterfall
			:cardsData="cardsData"
			@moreData="getUserPost"
			:nopost="nopost"
		></waterfall>

		<v-dialog v-model="changeInfoWin" max-width="800">
			<v-card>
				<v-col cols="10" offset="1">
					<v-card-title>修改资料</v-card-title>
					<v-card-text>
						<v-text-field v-model="cUserName" label="用户昵称"></v-text-field>
						<v-text-field v-model="cMail" label="邮箱"></v-text-field>
						<v-textarea v-model="cIntro" label="个人简介" outlined></v-textarea>
					</v-card-text>
					<v-card-actions>
						<v-spacer></v-spacer>
						<v-btn outlined color="error" large @click="changeInfoWin = false"
							>放弃</v-btn
						>
						<v-btn
							color="primary"
							large
							@click="commitChange"
							:disabled="btndis"
							>修改</v-btn
						>
					</v-card-actions>
				</v-col>
			</v-card>
		</v-dialog>
		<v-dialog v-model="changeBannarWin" max-width="800">
			<v-card>
				<v-col cols="10" offset="1">
					<v-card-title>更换背景图</v-card-title>
					<v-card-text>
						<v-file-input
							prepend-icon="mdi-image"
							accept="image/*"
							label="点击上传新背景图"
							v-model="cBannar"
						></v-file-input>
					</v-card-text>
					<v-card-actions>
						<v-spacer></v-spacer>
						<v-btn outlined color="error" large @click="changeBannarWin = false"
							>放弃</v-btn
						>
						<v-btn
							color="primary"
							large
							@click="commitBannar"
							:disabled="btndis"
							>修改</v-btn
						>
					</v-card-actions>
				</v-col>
			</v-card>
		</v-dialog>
		<v-dialog v-model="changeAvatarWin" max-width="800">
			<v-card>
				<v-col cols="10" offset="1">
					<v-card-title>更换头像</v-card-title>
					<v-card-text>
						<v-file-input
							prepend-icon="mdi-account-circle"
							accept="image/*"
							label="点击上传新头像"
							v-model="cAvatar"
						></v-file-input>
					</v-card-text>
					<v-card-actions>
						<v-spacer></v-spacer>
						<v-btn outlined color="error" large @click="changeAvatarWin = false"
							>放弃</v-btn
						>
						<v-btn
							color="primary"
							large
							@click="commitAvatar"
							:disabled="btndis"
							>修改</v-btn
						>
					</v-card-actions>
				</v-col>
			</v-card>
		</v-dialog>

		<v-snackbar v-model="resultWin">
			{{ result }}
			<template v-slot:action="{ attrs }">
				<v-btn color="blue" text v-bind="attrs" @click="resultWin = false">
					知道了
				</v-btn>
			</template>
		</v-snackbar>
	</div>
</template>

<script>
import waterfall from "@/components/WaterFall.vue";
export default {
	components: {
		waterfall,
	},
	data() {
		return {
			changeInfoWin: false,
			changeBannarWin: false,
			changeAvatarWin: false,
			cUserName: "",
			cMail: "",
			cIntro: "",
			cAvatar: null,
			cBannar: null,
			cardsData: [],
			btndis: false,
			result: "",
			resultWin: false,
			nopost: false,
		};
	},
	methods: {
		//提交头像的更换
		commitAvatar() {
			this.btndis = true;
			let formData = new FormData();
			formData.append("img", this.cAvatar);
			this.axios
				.post("uploadPics", formData, {
					headers: { "Content-Type": "multipart/form-data" },
				})
				.then((res) => {
					if (res.status != 200) {
						this.result = "上传头像失败，请重试";
						this.resultWin = true;
					} else {
						this.axios
							.get("changeAvatar", {
								params: { url: res.data },
							})
							.then((res) => {
								if (res.status == 200) {
									this.changeAvatarWin = false;
									this.result = res.data;
									this.resultWin = true;
									this.btndis = false;
									this.checkLogin();
								}
							})
							.catch((res) => {
								this.btndis = false;
								this.result = res.response.data;
								this.resultWin = true;
							});
					}
				});
		},
		//提交bannar的更换
		commitBannar() {
			this.btndis = true;
			let formData = new FormData();
			formData.append("img", this.cBannar);
			this.axios
				.post("uploadPics", formData, {
					headers: { "Content-Type": "multipart/form-data" },
				})
				.then((res) => {
					if (res.status != 200) {
						this.result = "上传bannar失败，请重试";
						this.resultWin = true;
					} else {
						this.axios
							.get("changeBannar", {
								params: { url: res.data },
							})
							.then((res) => {
								if (res.status == 200) {
									this.checkLogin();
									this.changeBannarWin = false;
									this.result = res.data;
									this.resultWin = true;
									this.btndis = false;
								}
							})
							.catch((res) => {
								this.btndis = false;
								this.result = res.response.data;
								this.resultWin = true;
							});
					}
				});
		},
		//向后端提交用户信息的修改
		commitChange() {
			this.btndis = true;
			let sendData = {
				name: this.cUserName,
				mail: this.cMail,
				intro: this.cIntro,
			};
			this.axios
				.post("changeInfo", sendData)
				.then((res) => {
					if (res.status == 200) {
						this.changeInfoWin = false;
						this.result = res.data;
						this.resultWin = true;
						this.btndis = false;
						this.checkLogin();
					}
				})
				.catch((res) => {
					this.btndis = false;
					this.result = res.response.data;
					this.resultWin = true;
				});
		},
		//用户点击更改资料按钮
		changeInfo() {
			this.cUserName = this.userData.name;
			this.cMail = this.userData.mail;
			this.cIntro = this.userData.intro;
			this.changeInfoWin = true;
		},

		//获取自己发送的post，懒加载
		getUserPost(num) {
			this.axios
				.get("getPostByUser", {
					params: { name: this.$store.state.userData.name, num: num },
				})
				.then((res) => {
					if (res.status == 200) {
						if (res.data.posts == undefined) {
							this.nopost = true;
							return;
						}
						res.data.posts.forEach((post) => {
							this.cardsData.push(post);
						});
					}
				});
		},

		//从cookie中读取数据，检测是否已登录
		checkLogin() {
			this.axios.get("getUserInfoByCookie").then((res) => {
				if (res.status == 200) {
					this.$store.commit("setUserData", res.data);
				}
			});
		},
	},
	mounted() {
		const that = this;
		window.onresize = () => {
			return (() => {
				window.screenWidth = document.body.clientWidth;
				that.screenWidth = window.screenWidth;
			})();
		};
		this.getUserPost(0);
	},

	watch: {
		refresh() {
			this.getUserPost(0);
		},
	},

	computed: {
		userData() {
			return this.$store.state.userData;
		},
		refresh() {
			return this.$store.state.refreshFlag;
		},
	},
};
</script>

<style scoped>
.v-btn {
	margin: 0 0.5rem;
}
.avatar {
	height: 150px;
	width: 150px;
	margin-top: -50px;
	margin-left: 50px;
	margin-bottom: 20px;
	border-radius: 100% !important;
	box-shadow: 0px 0px 10px 5px #fff;
}
.changeBtn {
	right: 10px;
	position: absolute;
	bottom: 5px;
}
.count-text {
	font-size: 1.5rem;
	font-weight: 700;
	color: #686868;
	padding-left: 5px;
}
.colum-line {
	border-right: 2px #d1d1d1 solid;
	width: auto;
	width: max-content;
	padding: 15px;
}
.fall-card {
	position: relative;
	width: 30%;
	margin: 1%;
	overflow: hidden;
	height: auto;
}
.changeBtnTop {
	right: 10px;
	position: absolute;
	top: 5px;
	opacity: 0.7;
	transition: 1s;
}
.changeBtnTop:hover {
	opacity: 1;
	transition: 0.5s;
}
.changAvatarBtn {
	height: 200px;
	margin: 0;
	opacity: 0;
	transition: opacity 0.5s;
}
.changAvatarBtn:hover {
	opacity: 1;
}
.bannar {
	height: 300px;
}
</style>