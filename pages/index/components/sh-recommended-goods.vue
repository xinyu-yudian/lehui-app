<template>
	<!-- 为你推荐 -->
	<view class="hot-goods u-m-b-10 u-p-x-16">
		<view class="content-line">---- <text class="content-title">为你推荐</text> ----</view>
		<view class="u-waterfall">
			<view id="u-left-column" class="u-column">
				<view class="goods-item u-m-b-16 u-flex u-row-center u-col-center" v-for="leftGoods in leftList" :key="leftGoods.id">
					<shopro-goods-card
						:detail="leftGoods"
						:type="leftGoods.activity_type"
						:image="leftGoods.image"
						:title="leftGoods.title"
						:subtitle="leftGoods.subtitle"
						:price="leftGoods.price"
						:originPrice="leftGoods.original_price"
						:sales="leftGoods.sales"
						:tagTextList="leftGoods.activity_discounts_tags"
						@click="$Router.push({ path: '/pages/goods/detail', query: { id: leftGoods.id } })"
					></shopro-goods-card>
				</view>
			</view>
			<view id="u-right-column" class="u-column">
				<view class="goods-item  u-m-b-16 u-flex u-row-center u-col-center" v-for="rightGoods in rightList" :key="rightGoods.id">
					<shopro-goods-card
						:detail="rightGoods"
						:type="rightGoods.activity_type"
						:image="rightGoods.image"
						:title="rightGoods.title"
						:subtitle="rightGoods.subtitle"
						:price="rightGoods.price"
						:originPrice="rightGoods.original_price"
						:sales="rightGoods.sales"
						:tagTextList="rightGoods.activity_discounts_tags"
						@click="$Router.push({ path: '/pages/goods/detail', query: { id: rightGoods.id } })"
					></shopro-goods-card>
				</view>
			</view>
		</view>
		<!-- m -->
	</view>
</template>

<script>
/**
 * 自定义之为你推荐
 * @property {Object} detail - 推荐商品信息
 */
import shGoodsCard from './sh-goods-card.vue';
export default {
	components: {
		shGoodsCard
	},
	data() {
		return {
			listParams: {
				page: 1
			}, //当前分页
			lastPage: 1, //总分页
			total: 0, //总商品数
			perPage: 0, //单页商品数
			goodsList: [],
			isRefresh: false,

			// 瀑布流 350-330
			addTime: 100, //排序间隙时间
			leftHeight: 0,
			rightHeight: 0,
			leftList: [],
			rightList: [],
			tempList: [],
			goodsType: this.detail.style // 商品模板
		};
	},

	props: {
		detail: {
			type: String,
			default: () => {}
		}
	},
	created() {
		if (this.detail) {
			this.listParams.goods_ids = this.detail;
			this.getGoodsList();
		}
		// if (this.detail.ids) {
		// 	this.listParams.goods_ids = this.detail;
		// 	console.log(this.detail.ids);
		// 	this.getGoodsList();
		// }
	},
	methods: {
		// 瀑布流相关
		async splitData() {
			if (!this.tempList.length) return;
			let item = this.tempList[0];
			if (!item) return;

			// 分左右
			if (this.leftHeight < this.rightHeight) {
				this.leftHeight += item.activity_discounts_tags?.length ? 350 : 330;
				this.leftList.push(item);
			} else if (this.leftHeight > this.rightHeight) {
				this.rightHeight += item.activity_discounts_tags?.length ? 350 : 330;
				this.rightList.push(item);
			} else {
				this.leftHeight += item.activity_discounts_tags?.length ? 350 : 330;
				this.leftList.push(item);
			}

			// 移除临时列表的第一项，如果临时数组还有数据，继续循环
			this.tempList.splice(0, 1);
			if (this.tempList.length) {
				setTimeout(() => {
					this.splitData();
				}, this.addTime);
			}
		},
		clear() {
			this.leftList = [];
			this.rightList = [];
			this.leftHeight = 0;
			this.rightHeight = 0;
			this.tempList = [];
		},

		// 商品列表
		getGoodsList() {
			let that = this;
			that.$http('goods.recommendLists', this.listParams).then(res => {
				if (res.code === 1) {
					this.lastPage = res.data.last_page;
					this.total = res.data.total;
					this.perPage = res.data.per_page;
					this.isRefresh = false;
					that.goodsList = [...that.goodsList, ...res.data.data];
					that.tempList = res.data.data;
					that.goodsList.length && that.splitData();
				}
			});
		},

		// 加载更多
		loadMore() {
			if (!this.isRefresh) {
				// 加载更多
				if (this.listParams.page < this.lastPage) {
					this.isRefresh = true;
					this.listParams.page += 1;
					this.getGoodsList();
				} else {
					// 重置为1页数据
					this.listParams.page = 1;
					this.lastPage = 1;
					this.goodsList = [];
					this.clear();
					this.getGoodsList();
				}
			}
		}
	}
};
</script>

<style lang="scss">
@mixin vue-flex($direction: row) {
	/* #ifndef APP-NVUE */
	display: flex;
	flex-direction: $direction;
	/* #endif */
}
.u-waterfall {
	@include vue-flex;
	flex-direction: row;
	align-items: flex-start;
}

.u-column {
	@include vue-flex;
	flex: 1;
	flex-direction: column;
	height: auto;
}

.u-image {
	width: 100%;
}
// 为你推荐
.hot-goods {
	background: none;
	.content-line{
		color: #999999;
		width: 100%;
		text-align: center;
		height: 60rpx;
		line-height: 60rpx;
		.content-title{
			color: darkcyan;
			padding: 0 30rpx;
		}
	}
	
	.refresh-btn {
		margin-left: 50%;
		transform: translateX(-50%);
		width: 156rpx;
		line-height: 50rpx;
		background: #ffffff;
		border-radius: 25rpx;
		font-size: 22rpx;
		font-weight: 500;
		color: #999999;
		white-space: nowrap;
	}
	.refresh-active {
		transform: rotate(360deg);
		transition: all linear 0.5s;
	}
}
</style>
