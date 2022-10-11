<template>
	<view>
		<view class="outview">

			<view class="wrap" v-for="lottery in lotteryList" :key="lottery.id" @click="$Router.push({ path: '/pages/user/lottery/listdetaill', query: { id: lottery.id } })">
				<u-row gutter="16" justify="space-between">
					<u-col span="5">
						<image style="height:220rpx;width: 230rpx;" mode="aspectFill" :src=lottery.image />
					</u-col>
					<u-col span="7">
						<view class="u-body-item-title u-line-2">{{lottery.title}}</view>
						<view class="demo-layout " style="margin-top: 10rpx;">
							<u-count-down v-if="lottery.ifstart==0" separator="zh" :timestamp=lottery.timecount :show-border="true" font-size="24" color="red"
								separator-color="red" border-color="#e0e0eb">
							   
							</u-count-down>
						</view>
						<template v-if="lottery.ifstart==0">
							<view class="demo-layout " v-if="lottery.isJoin == 1">
								<u-button size="medium" disabled type="error">等待开奖</u-button>
							</view>
							<view class="demo-layout " v-else>
								<u-button @click="$Router.push({ path: '/pages/user/lottery/listdetaill', query: { id: lottery.id } })" size="medium" type="error">参与抽奖</u-button>
							</view>
						</template>
						<template v-else>
							<view class="demo-layout ">
								<u-button size="medium" disabled type="error">已开奖</u-button>
							</view>
						</template>
						
						

					</u-col>
				</u-row>
			</view>

		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				timestamp: 8640000,
				lotteryList: [],
			};
		},
		onShow() {
			this.getLotteryList();
		},
		methods: {
			getLotteryList() {
				this.$http('lottery.lotteryList').then(res => {
					console.log('我返回了')
					if (res.code === 1) {
						this.lotteryList = res.data;
						uni.setStorageSync('lotteryTime',res.data[0].updatetime)
					}
				});
			}
		}

	};
</script>

<style scoped lang="scss">
	.outview {
		width: auto;
		height: auto;
		margin: 20rpx;

		border-radius: 10rpx;
	}

	.wrap {
		padding: 24rpx;
		margin-top: 25rpx;
		background-color: #fff;
	}

	.u-row {
		margin: 40rpx 0;
	}

	.demo-layout {
		height: 80rpx;
		border-radius: 8rpx;
	}

	.bg-purple {
		background: #d3dce6;
	}

	.bg-purple-light {
		background: #e5e9f2;
	}

	.bg-purple-dark {
		background: #99a9bf;
	}
</style>
