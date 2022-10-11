<template>
	<view>
		<LuckyWheel ref="myLucky" width="600rpx" height="600rpx" :blocks="blocks" :prizes="prizes" :buttons="buttons"
			:defaultStyle="defaultStyle" @start="startCallBack" @end="endCallBack" />
	</view>
</template>

<script>
	import LuckyWheel from '@lucky-canvas/uni/lucky-wheel'
	export default {
		components: {
			LuckyWheel
		},
		data() {
			return {
				blocks: [{
					padding: '13px',
					background: '#617df2'
				}],
				prizes: [{
						fonts: [{
							text: '0',
							top: '10%'
						}],
						background: '#e9e8fe'
					},
					{
						fonts: [{
							text: '1',
							top: '10%'
						}],
						background: '#b8c5f2'
					},
					{
						fonts: [{
							text: '2',
							top: '10%'
						}],
						background: '#e9e8fe'
					},
					{
						fonts: [{
							text: '3',
							top: '10%'
						}],
						background: '#b8c5f2'
					},
					{
						fonts: [{
							text: '4',
							top: '10%'
						}],
						background: '#e9e8fe'
					},
					{
						fonts: [{
							text: '5',
							top: '10%'
						}],
						background: '#b8c5f2'
					},
				],
				buttons: [{
						radius: '50px',
						background: '#617df2'
					},
					{
						radius: '45px',
						background: '#afc8ff'
					},
					{
						radius: '40px',
						background: '#869cfa',
						pointer: true,
						fonts: [{
							text: '开始\n抽奖',
							top: '-20px'
						}]
					},
				],
			}
		},
		methods: {
			// 点击抽奖按钮触发回调
			startCallBack() {
				// 先开始旋转
				this.$refs.myLucky.play()
				// 使用定时器来模拟请求接口
				setTimeout(() => {
					// 假设后端返回的中奖索引是0
					const index = 0
					// 调用stop停止旋转并传递中奖索引
					this.$refs.myLucky.stop(index)
				}, 3000)
			},
			// 抽奖结束触发回调
			endCallBack(prize) {
				// 奖品详情
				console.log(prize)
			}
		}
	}
</script>
