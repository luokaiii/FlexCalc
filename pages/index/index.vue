<template>
	<view class="container">
		<view>
			多功能智算小算盘，算清房贷车贷汇率账，预知延迟退休新时光。
		</view>
		<uni-grid :column="3" :highlight="true" :showBorder="false" @change="clickItem">
			<uni-grid-item v-for="(item,index) in gridList" :index="index" :key="index">
				<view class="grid-item-box" style="background-color: #fff;" @click="gotoNextPage(item.link)">
					<image :src="item.cover" mode="heightFix" style="height: 60rpx; margin-bottom: 20rpx;"></image>
					<text class="text">{{item.text}}</text>
				</view>
			</uni-grid-item>
		</uni-grid>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				gridList: [{
					cover: '/static/home/延迟退休.png',
					link: '/pages/tuixiu/tuixiu',
					text: '法定退休计算器'
				}, {
					cover: '/static/home/房贷.png',
					link: '',
					text: '房贷计算器'
				}, {
					cover: '/static/home/车贷.png',
					link: '',
					text: '车贷计算器'
				}, {
					cover: '/static/home/个税.png',
					link: '',
					text: '个税计算器'
				}, {
					cover: '/static/home/汇率.png',
					link: '',
					text: '汇率计算器'
				}]
			}
		},
		onShareAppMessage() {
			return {
				title: '多功能智算小算盘，算清房贷车贷汇率账，预知延迟退休新时光。',
				path: '/pages/index/index'
			}
		},
		methods: {
			gotoNextPage(url) {
				uni.navigateTo({
					url: url,
					animationType: 'fade-in'
				})
			}
		}
	}
</script>

<style>
	.container {
		padding: 20px;
		font-size: 14px;
		line-height: 24px;
	}

	.grid-item-box {
		flex: 1;
		// position: relative;
		/* #ifndef APP-NVUE */
		display: flex;
		/* #endif */
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding: 15px 0;
	}
</style>