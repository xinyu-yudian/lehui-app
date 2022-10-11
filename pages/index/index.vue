<!-- 首页 -->
<template>
	<view class="home-wrap u-m-b-20">
		<!-- 空白页 -->
		<!-- #ifdef APP-PLUS -->
		<u-no-network @retry="init"></u-no-network>
		<!-- #endif -->
		<shopro-empty v-if="!hasTemplate" :image="$IMG_URL + '/imgs/empty/template_empty.png'" tipText="暂未找到模板，请前往装修~">
		</shopro-empty>

		<view v-else-if="isConnected && isRefresh" class="content-box">
			<!-- 导航栏 -->
			<home-head v-if="headSwiperList && headSwiperList.length" :isScorll="isScorll" borderRadius="0"
				:navTitle="initShop.name" :list="headSwiperList"></home-head>
			<!-- 自定义模块 -->
			<view class="template-box">
				<block v-for="(item, index) in homeTemplate" :key="item.id">
					<!-- 轮播 -->
					<sh-banner v-if="item.type === 'banner' && index !== 0" :Px="item.content.x" :Py="item.content.y"
						:borderRadius="item.content.radius" :height="item.content.height" :list="item.content.list">
					</sh-banner>

					<!-- 搜索 -->
					<sh-search v-if="item.type === 'search'"></sh-search>

					<!-- 滑动宫格 -->
					<sh-grid-swiper v-if="item.type === 'menu'" :list="item.content.list"
						:oneRowNum="item.content.style"></sh-grid-swiper>

					<!-- 推荐商品 -->
					<sh-hot-goods v-if="item.type === 'goods-list' || item.type === 'goods-group'"
						:detail="item.content"></sh-hot-goods>
					<!-- 广告魔方 -->
					<sh-adv v-if="item.type === 'adv'" :detail="item.content"></sh-adv>
					<!-- 优惠券 -->
					<sh-coupon v-if="item.type === 'coupons'" :detail="item.content"></sh-coupon>
					<!-- 秒杀-->
					<sh-seckill v-if="item.type === 'seckill'" :detail="item.content"></sh-seckill>
					<!-- 拼团 -->
					<sh-groupon v-if="item.type === 'groupon'" :detail="item.content"></sh-groupon>
					<!-- 富文本 -->
					<sh-richtext v-if="item.type === 'rich-text'" :richId="item.content.id"></sh-richtext>
					<!-- 功能标题 -->
					<sh-title-card v-if="item.type === 'title-block'" :title="item.content.name"
						:bgImage="item.content.image" :titleColor="item.content.color"></sh-title-card>
					<!-- 直播 -->
					<!-- #ifdef MP-WEIXIN -->
					<sh-live v-if="item.type === 'live' && HAS_LIVE" :detail="item.content"></sh-live>
					<!-- #endif -->
				</block>
			</view>

			<!-- 分类选项卡 -->
			<sh-category-tabs
				v-if="categoryTabsData && categoryTabsData.category_arr && categoryTabsData.category_arr.length"
				:enable="enable" :styleType="categoryTabsData.style" :tabsList="categoryTabsData.category_arr">
			</sh-category-tabs>
			<!-- 登录提示 -->
			<shopro-auth-modal> </shopro-auth-modal>
			<!-- 悬浮按钮 -->
			<shopro-float-btn></shopro-float-btn>
			<!-- 连续弹窗提醒 -->
			<shopro-notice-modal v-if="!showPrivacy && isLogin"></shopro-notice-modal>
			<!-- 未参加的抽奖活动提示 -->
			
			<view>
				<!-- <view>
					<button @click="showDiv()">出来吧，我的弹窗！</button>
				</view> -->
				<view :hidden="userFeedbackHidden" class="popup_content">
					<view>
						<uni-card :isShadow="true" mode="basic" extra="技术没有上限" @click="clickCard">
							<view @click="clickPopup">
								<view>
									<image  mode="aspectFit"
										:src="rdata" />
								</view>
								<!-- <view class="content-box"><text style="font-size: 12px;">标题卡片带有一个双标题头部，右侧为额外描述信息
										，内容可自定义实现</text></view> -->
							</view>
						</uni-card>
					</view>
				</view>
				<view class="popup_overlay" :hidden="userFeedbackHidden" @click="hideDiv()"> </view>
			</view>
			<!-- 隐私协议 -->
			<!-- #ifdef APP-PLUS -->
			<privacy-modal v-if="initShop && initShop.name" v-model="showPrivacy"></privacy-modal>
			<!-- #endif -->
			<!-- #ifdef H5 -->
			<view class="tabbar-hack" style="height: 120rpx; width:100%;"></view>
			<!-- #endif -->
		</view>
		<shopro-tabbar></shopro-tabbar>
	</view>
</template>

<script>
	import shBanner from './components/sh-banner.vue';
	import shGridSwiper from './components/sh-grid-swiper.vue';
	import shHotGoods from './components/sh-hot-goods.vue';
	import shAdv from './components/sh-adv.vue';
	import shCoupon from './components/sh-coupon.vue';
	import shSeckill from './components/sh-seckill.vue';
	import shGroupon from './components/sh-groupon.vue';
	import shRichtext from './components/sh-richtext.vue';
	import shTitleCard from './components/sh-title-card.vue';
	import shSearch from './components/sh-search.vue';
	import shCategoryTabs from './components/sh-category-tabs.vue';

	import privacyModal from './index/privacy-modal.vue';
	import homeHead from './index/home-head.vue';

	// #ifdef MP-WEIXIN
	import {
		HAS_LIVE
	} from '@/env';
	import shLive from './components/sh-live.vue';
	// #endif

	import {
		mapMutations,
		mapActions,
		mapState,
		mapGetters
	} from 'vuex';
	export default {
		components: {
			shBanner,
			shGridSwiper,
			shGroupon,
			shHotGoods,
			shAdv,
			shCoupon,
			shSeckill,
			shRichtext,
			shTitleCard,
			shSearch,
			shCategoryTabs,

			privacyModal,
			homeHead,

			// #ifdef MP-WEIXIN
			shLive
			// #endif
		},
		data() {
			return {
				// #ifdef MP-WEIXIN
				HAS_LIVE: HAS_LIVE,
				// #endif
				isRefresh: true,

				enable: false, //是否开启吸顶。
				isConnected: true, //是否有网
				showPrivacy: false, //协议
				isScorll: false,
				userFeedbackHidden: true,
				rdata: '',
				feedbackContent: ''

			};
		},
		computed: {
			...mapGetters(['initShop', 'homeTemplate', 'hasTemplate', 'isLogin']),
			// 头部模块数据
			headSwiperList() {
				if (this.homeTemplate?.length) {
					return this.homeTemplate[0]?.content?.list;
				}
			},
			// 分类选项卡数据
			categoryTabsData() {
				if (this.homeTemplate?.length) {
					return this.homeTemplate[this.homeTemplate.length - 1]?.content;
				}
			}
		},
		onPullDownRefresh() {
			this.init();
		},
		onPageScroll(e) {
			this.isScorll = e.scrollTop > 100 ? true : false;
		},
		
		onShow() {
			if (this.isLogin) { //如果登录，弹出没参加过的活动图片。如果没登录也弹,不过要调另一个接口了
				this.$http('lottery.needJoin').then(res => {
					if (res.code === 1 && res.data != null) {
						this.rdata = 'https://lehuicc.oss-cn-guangzhou.aliyuncs.com' + res.data.image
						if(uni.getStorageSync('lotteryTime')==res.data.updatetime){
							this.userFeedbackHidden = true
						}else{
							this.userFeedbackHidden = false
						}
					}
				});
			}else{
				this.$http('lottery.needJoinNologin').then(res => {
					console.log(res.data)
					if (res.code === 1 && res.data != null) {
						this.rdata = 'https://lehuicc.oss-cn-guangzhou.aliyuncs.com' + res.data.image
						this.userFeedbackHidden = false
					}
				});
			}
			this.enable = true;
			this.isLogin && this.getCartList();
			// 网络变化检测
			uni.onNetworkStatusChange(res => {
				this.isConnected = res.isConnected;
				res.isConnected && this.init();
			});
		},
		onHide() {
			this.enable = false;
		},
		onLoad() {
			this.test1()
			// #ifdef APP-VUE
			// app隐私协议弹窗
			if (!plus.runtime.isAgreePrivacy()) {
				this.showPrivacy = true;
				this.showNoticeModal = false;
			}
			// #endif
		},
		methods: {
			...mapActions(['appInit', 'getTemplate', 'getCartList']),
			// 初始化
			init() {
				this.isRefresh = false;
				return Promise.all([this.getTemplate()]).then(() => {
					uni.stopPullDownRefresh();
					this.isRefresh = true;
				});
			},
			clickPopup(){
				if(this.isLogin){
					this.$Router.push({ path: '/pages/user/lottery/list' })
				}else{
					this.$store.dispatch('showAuthModal', 'accountLogin');
				}
				
			},
			
			
			
			test1() {
				
			},
			showDiv() {
				this.userFeedbackHidden = false
			},
			hideDiv() {
				this.userFeedbackHidden = true
			},
			submitFeedback() {
				this.userFeedbackHidden = true

			}

		},
		mounted:function(){
			
			if(!this.isLogin){
				this.$store.dispatch('showAuthModal', 'accountLogin');
			}
		}
	};
</script>

<style lang="scss">
	.img-box {
		width: 50rpx;
	}

	.popup_overlay {
		position: fixed;
		top: 0%;
		left: 0%;
		width: 100%;
		height: 100%;
		background-color: black;
		z-index: 1001;
		-moz-opacity: 0.8;
		opacity: .80;
		filter: alpha(opacity=88);
	}

	.popup_content {
		position: fixed;
		top: 50%;
		left: 50%;
		padding: 10rpx;
		//width: 580rpx;
		//height: 520rpx;
		//margin-left: -270rpx;
		//margin-top: -270rpx;
		transform: translate(-50%,-50%);
		border: 0px solid white;
		border-radius: 12rpx;
		background-color: transparent;
		z-index: 1002;
		overflow: auto;
		display: inline-block;
	}
	.popup_content image{
		width: 550rpx;
		height: 800rpx;
	}

	.popup_title {
		width: 480rpx;
		text-align: center;
		font-size: 32rpx;
	}

	.popup_textarea_item {
		padding-top: 5rpx;
		height: 50rpx;
		width: 440rpx;
		background-color: #F1F1F1;
		margin-top: 20rpx;
		margin-left: 20rpx;
		padding-top: 25rpx;
	}

	.popup_textarea {
		width: 410rpx;
		font-size: 26rpx;
		margin-left: 20rpx;
	}

	.popup_button {
		color: #000000;
	}

	.buttons {
		text-align: center;
		font-size: 50rpx;
		margin-top: 40rpx;
		background-color: #007AFF;
	}
</style>
