<template>
	<view class="my-settle-container">

		<label class="radio" @click="changeAllState">
			<radio color="#c00000" :checked="isFullCheck" /><text>全选</text>
		</label>

		<view class="amount-box">
			合计：<text class="amount">￥{{checkedGoodsAmount}}</text>
		</view>

		<view class="btn-settle" @click="settlement">
			结算（{{checkedCount}}）
		</view>
	</view>
</template>

<script>
	import {
		mapGetters,
		mapMutations,
		mapState
	} from 'vuex'
	export default {
		name: "my-settle",
		data() {
			return {
				seconds: 3,
				timer: null,
			};
		},
		computed: {
			...mapGetters('m_cart', ['checkedCount', 'total', 'checkedGoodsAmount']),
			...mapGetters('m_user', ['addstr']),
			...mapState('m_user', ['token']),
			...mapState('m_cart',['cart']),
			isFullCheck() {
				return this.total === this.checkedCount
			}
		},
		methods: {
			...mapMutations('m_cart', ['updateAllGoodsState']),
			...mapMutations('m_user', ['updateRedirectInfo']),
			changeAllState() {
				this.updateAllGoodsState(!this.isFullCheck)
			},
			settlement() {
				if (!this.checkedCount) return uni.$showMsg('请选择要结算的商品!')

				if (!this.addstr) return uni.$showMsg('请选择收货地址!')

				if (!this.token) return this.delayNav()
				this.payOrder()
			},
			showTips(n) {
				uni.showToast({
					icon: 'none',
					title: '请登陆后再结算！' + n + '秒后跳转到登陆页面',
					mask: true,
					duration: 1500
				})
			},
			delayNav() {
				this.seconds = 3
				this.showTips(this.seconds)


				this.timer = setInterval(() => {
					this.seconds--

					if (this.seconds <= 0) {
						clearInterval(this.timer)

						uni.switchTab({
							url: '/pages/my/my',
							success: () => {
								this.updateRedirectInfo({
									openType: 'switchTab',
									from: '/pages/cart/cart'
								})
							}
						})
					}

					this.showTips(this.seconds)
				}, 1000)
			},
			async payOrder() {
				const orderInfo = {
					order_price: 0.01,
					consignee_addr: this.addstr,
					goods: this.cart.filter(x => x.goods_state).map(x => ({
						goods_id: x.goods_id,
						goods_number: x.goods_count,
						goods_price: x.goods_price
					}))
				}
				const {data:res} =await uni.$http.post('/api/public/v1/my/orders/create',orderInfo)
				console.log('订单信息',res)
				if(res.meta.status !== 200) return uni.$showMsg('创建订单失败!')
				const orderNumber = res.message.order_number
				
			}
		}
	}
</script>

<style lang="scss">
	.my-settle-container {
		position: fixed;
		bottom: 0;
		left: 0;
		width: 100%;
		height: 50px;
		background-color: white;
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding-left: 5px;
		font-size: 14px;

		.radio {
			display: flex;
			align-items: center;
		}

		.amount {
			color: #c00000;

		}

		.btn-settle {
			height: 50px;
			min-width: 100px;
			background-color: #c00000;
			color: white;
			line-height: 50px;
			text-align: center;
			padding: 0 10px;
		}
	}
</style>
