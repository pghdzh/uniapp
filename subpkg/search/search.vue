<template>
	<view>
		<view class="search-box">
			<uni-search-bar @input="input" :radius="100" cancelButton="none" focus="always" />
		</view>
		<view class="history-box" v-if="searchResults.length === 0">
			<view class="history-title">
				<text>搜索历史</text>
				<uni-icons type="trash" size="17" @click="clearHis"></uni-icons>
			</view>
			<view class="history-list">
				<uni-tag :text="item" v-for="(item,i) in historys" :key="i" customStyle="margin-right:5px; " @click="gotoGoodsList(item)"></uni-tag>
			</view>
		</view>
		<view class="sugg-list" v-else>
			<view class="sugg-item" v-for="(item,i) in searchResults" :key="i" @click="gotoGoodsList(item)">
				<view class="goods-name">{{item.goods_name}}</view>
				<uni-icons type="arrowright" size="16"></uni-icons>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				timer: null,
				kw: '',
				searchResults: [],
				historyList: []
			};
		},
		onLoad() {
			this.historyList = JSON.parse(uni.getStorageSync('kw') || '[]')
		},
		computed:{
			historys(){
				return [...this.historyList].reverse()
			}
		},
		methods: {
			input(e) {
				clearTimeout(this.timer)
				this.timer = setTimeout(() => {
					this.kw = e
					this.getSearchList()
				}, 500)
			},
			async getSearchList() {
				if (this.kw.length === 0) {
					this.searchResults = []
					return
				}
				const {
					data: res
				} = await uni.$http.get('/api/public/v1/goods/qsearch', {
					query: this.kw
				})
				if (res.meta.status !== 200) return uni.$showMsg()
				this.searchResults = res.message
				
				this.saveSearchHistory()
			},
			gotoDetail(item) {
				uni.navigateTo({
					url: '../goods_detail/goods_detail?goods_id=' + item.goods_id
				})
			},
			saveSearchHistory(){
				const set = new Set(this.historyList)
				set.delete(this.kw)
				set.add(this.kw)
				this.historyList = Array.from(set)
				uni.setStorageSync('kw',JSON.stringify(this.historyList))
			},
			clearHis(){
				this.historyList=[]
				uni.setStorageSync('kw','[]')
			},
			gotoGoodsList(kw){
				uni.navigateTo({
					url:'/subpkg/goods_list/goods_list?query=' + kw
				})
			}
		}
	}
</script>

<style lang="scss">
	.uni-searchbar {
		background-color: #C00000;
	}

	.search-box {
		position: sticky;
		top: 0;
		z-index: 99;
	}
	
	.history-box{
		padding: 0 5px;
		.history-title{
			display: flex;
			justify-content: space-between;
			height: 40px;
			align-items: center;
			font-size: 13px;
			border-bottom: 1px solid #efefef;
		}
		.history-list{
			display: flex;
			flex-wrap: wrap;
			margin-top: 5px;
		}
	}
		
	.sugg-list {
		padding: 0 5px;

		.sugg-item {
			font-size: 12px;
			padding: 13px 0;
			border-bottom: 1px solid #efefef;
			display: flex;
			align-items: center;
			justify-content: space-between;

			.goods-name {
				white-space: nowrap;
				overflow: hidden;
				text-overflow: ellipsis;
				margin-right: 3px;
			}
		}
	}
</style>
