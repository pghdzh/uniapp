<template>
	<view class="login-container">
		<uni-icons type="contact-filled" size="100" color="#AFAFAF"></uni-icons>
		<button type="primary" class="btn-login" @click="getUserProfile">一键登录</button>
		<view class="tips-text">
			登录后尽享更多权益
		</view>
	</view>
</template>

<script>
	import {
		mapMutations,mapState
	} from 'vuex'
	export default {
		name: "my-login",
		data() {
			return {

			};
		},
		computed:{
			...mapState('m_user',['redirectInfo'])
		},
		methods: {
			...mapMutations('m_user', ['upadteUserInfo', 'updateToken','updateRedirectInfo']),
			getUserProfile() {
				uni.getUserProfile({
					desc: '你的授权信息',
					success: (res) => {
						// 将信息存到 vuex 中
						this.upadteUserInfo(res.userInfo)
						this.getToken(res)
					},
					fail: (res) => {
						return uni.$showMsg('您取消了登录授权')
					}
				})
			},
			async getToken(info) {
				
				const [err, res] = await uni.login().catch(err => err)
				
				if (res.errMsg !== 'login:ok') return uni.$showMsg('登录失败！')

				this.updateToken(res.code)
				this.navBack()
			},
			navBack(){
				if(this.redirectInfo && this.redirectInfo.openType === 'switchTab'){
					uni.switchTab({
						url:this.redirectInfo.from,
						complete:() => {
							this.updateRedirectInfo(null)
						}
					})
				}
			}
		}
	}
</script>

<style lang="scss">
	.login-container {
		height: 750rpx;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		background-color: #f8f8f8;
		position: relative;
		overflow: hidden;

		&::after {
			content: '';
			display: block;
			position: absolute;
			width: 100%;
			height: 40px;
			left: 0;
			bottom: 0;
			background-color: white;
			border-radius: 100%;
			transform: translateY(50%);
		}

		.btn-login {
			width: 90%;
			border-radius: 100px;
			margin: 15px 0;
			background-color: #c00000;
		}

		.tips-text {
			font-size: 12px;
			color: gray;
		}
	}
</style>
