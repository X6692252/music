<template>
	<view class="container bg-main pos-r">
		<musichead title="注册" :icon="true" :iconBlack="true"></musichead>
		<view class="padding-xl dflex-c dflex-flow-c">
			<view class="portrait-box margin-bottom">
				<image class="headimg border-radius-c" :src="(member && member.member_headimg) || '/static/images/user/default.png'"></image>
			</view>
			<!-- 昵称 -->
			<view class="w-full dflex padding-bottom-sm">
				<view class="iconfont iconuser margin-right"></view>
				<view class="flex1 dflex">
					<input class="border-line padding-sm flex1" type="text" data-key="nickname" maxlength="15"
						:value="nickname" @input="inputChange" placeholder="请输入昵称" />
				</view>
			</view>
			<!-- 手机号 -->
			<view class="w-full dflex padding-bottom-sm">
				<view class="iconfont iconshouji margin-right"></view>
				<view class="flex1 dflex">
					<input class="border-line padding-sm flex1" type="number" data-key="mobile" maxlength="11"
						:value="mobile" @input="inputChange" placeholder="请输入手机号" />
					<view v-if="0 == 1" class="padding-tb-sm ft-dark">获取</view>
				</view>
			</view>
			<!-- 密码 -->
			<view class="w-full dflex padding-bottom-sm">
				<view class="iconfont iconmima margin-right"></view>
				<!-- #ifdef MP -->
				<input class="border-line padding-sm flex1" type="text" password data-key="password" maxlength="20"
					:value="password" @input="inputChange" placeholder="请输入密码" />
				<!-- #endif -->
				<!-- #ifndef MP -->
				<input class="border-line padding-sm flex1" type="password" password data-key="password" maxlength="20"
					:value="password" @input="inputChange" placeholder="请输入密码" />
				<!-- #endif -->

			</view>
			<!-- 验证码 -->
			<view class="w-full dflex padding-bottom-sm">
				<view class="iconfont iconyanzheng margin-right"></view>
				<view class="border-line flex1 dflex">
					<input class="padding-sm flex1" type="number" data-key="code" maxlength="6" :value="code"
						@input="inputChange" @confirm="register" placeholder="请输入验证码" />
					<view v-if="!is_send" class="padding-tb-sm ft-base" @click="sendCode">发送验证码</view>
					<view v-else class="padding-tb-sm ft-base">{{code_time}}s 重新获取</view>
				</view>
			</view>
			<view class="w-full margin-top-xl">
				<view class="dflex-b border-radius-lg">
					<view class="tac padding-tb-sm flex1 bg-base fs" @click="register">注册</view>
				</view>
			</view>
		</view>

		<use-copyright class="pos-f w-full" style="bottom: -30rpx"></use-copyright>
	</view>
</template>

<script>
	// 引入返回上一级、返回首页 头部组件
	import musichead from '@/components/musichead/musichead.vue';
	import { register, sendCode, sendCodeVerify } from '@/common/api.js';
	import {
		mapState
	} from 'vuex';

	export default {
		data() {
			return {
				env: {},
				is_show: false,
				
				nickname: '',
				mobile: '',
				password: '',
				code: '',

				is_register: false,
				is_send: false,
				code_time: 30,
				timer: 0,
			};
		},
		computed: {
			...mapState(["member"])
		},
		// 局部组件
		components: {
			musichead
		},
		// 页面加载获取 wx.login code
		onShow() {
			console.log('login Show');
			// #ifdef MP-WEIXIN
			let lopts = uni.getLaunchOptionsSync();
			// #endif
		},
		onLoad() {
			// this.$api.get_env((res) => {
			// 	this.env = res;
			// 	// console.log(this.env);
			// 	this.is_mp = this.env.is_mp;
			// 	this.platform = this.env.platform;
			// 	this.platform_icon = this.env.platform_icon;
			// 	this.platform_name = this.env.platform_name;
			// });
		},
		methods: {
			inputChange(e) {
				const key = e.currentTarget.dataset.key;
				this[key] = e.detail.value;
			},

			// 发送验证码
			sendCode() {
				let _this = this;

				if (!this.mobile) {
					this.$api.msg('请输入手机号');
					return;
				}
				if (!/(^1[3|4|5|7|8|9][0-9]{9}$)/.test(this.mobile)) {
					this.$api.msg('请输入正确的手机号码');
					return;
				}

				if (this.is_send) return;


				this.code_time = 30;
				
				// console.log(this.mobile);
				let ctcode = {
					phone: this.mobile
				}
				// 发送验证码
				sendCode(ctcode).then((res)=>{
					if (res.code == 200) {
						alert('验证码已发送');
						this.is_send = true;
					}
				});
			},
			tologin() {
				// 登录页
				uni.navigateBack({});
			},
			register() {
				let _this = this;
				
				if (_this.is_register) return;

				if (!this.mobile) {
					this.$api.msg('请输入手机号');
					return;
				}
				if (!/(^1[3|4|5|7|8|9][0-9]{9}$)/.test(this.mobile)) {
					this.$api.msg('请输入正确的手机号码');
					return;
				}

				if (!this.password) {
					this.$api.msg('请输入密码');
					return;
				}
				// if (this.$api.trim(this.password).length < 4) {
				// 	this.$api.msg('密码长度不能小于4位');
				// 	return;
				// }
				if (!this.code) {
					this.$api.msg('请输入验证码');
					return;
				}
				
				// #ifdef H5 || MP-360 || APP-PLUS
				uni.getUserProfile = (x) => {
					if (typeof x.success === 'function') {
						x.success({ userInfo: {} });
					}
				};
				// let verifyCode = {
				// 	phone: this.mobile,
				// 	captcha: this.code
				// };
				// console.log(verifyCode);
				let registerParams = {
					nickname: this.nickname,
					phone: this.mobile,
					password: this.password,
					captcha: this.code
				}
				console.log(registerParams);
				register(registerParams).then((res) => {
					console.log(res);
					if(res[1].statusCode === 200){
						console.log(res[1].data)
						uni.navigateTo({
							url: '/pages/login/login'
						});
					}
				})
				// #endif
			}

		},

	};
</script>

<style lang="scss">
	@import url("@/components/iconfont/iconfont.css");
	@import url("@/common/common.scss");
	page {
		background: #f4f4f4;
	}

	.container {
		width: 100vw;
		min-height: 100vh;
		overflow: hidden;
	}

	.portrait-box {
		image {
			width: 130rpx;
			height: 130rpx;
			border: 5rpx solid #fff;
		}
	}
</style>
