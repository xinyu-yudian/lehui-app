<template>
	<view class="scan">
		<video id="myVideo" :src="videoUrl" controls v-if="videoShoww"></video>
	</view>
</template>

<script>
	import { mapMutations, mapActions, mapState, mapGetters } from 'vuex';
	export default {
		data() {
			return {
				videoUrl:"",
				videoShoww:false,
				videoContext:'',
			}
		},
		computed: {
			...mapGetters(['initShop', 'userTemplate', 'isLogin', 'userInfo', 'authType']),
		},
		onShow() {
			
		},
		onHide(){
			if(this.videoContext != ''){
				this.videoContext.pause();
				this.videoContext = '';
			}
		},
		onLoad(option) {
			this.videoContext = '';
			let that = this;
			if (!this.isLogin) {
				uni.showToast({
					'title':'请先登录',
					'icon':'none'
				})
				setTimeout(function(){
					that.$Router.replaceAll('/pages/index/index'); 
				},3000)
				return;
			}
			console.log('qqqqqqqq');
			console.log(option);
			if(option && option.scene){
				let goods_id = option.scene;
				this.playvideo(goods_id);
			} else {
				console.log('无参数');
				this.openScan();
			}
		},
		methods: {
			playvideo(goods_id){
				let that = this;
				that.$http('goods.video', {
					id: goods_id
				}).then(res2 => {
					console.log('res2');
					console.log(res2);
					if (res2.code === 1) {
						that.videoUrl = res2.data;
						// that.videoUrl = 'http://clips.vorwaerts-gmbh.de/big_buck_bunny.mp4';
						// that.videoUrl = 'http://lehuicc.oss-cn-guangzhou.aliyuncs.com/uploads/20220828/1788ec1f213434ced14b881618d7081d.mov';
						if(that.videoUrl){
							that.videoShoww = true;
							that.videoContext = uni.createVideoContext('myVideo')
							that.videoContext.play()
						}else{
							uni.showModal({
								title:"错误!",
								showCancel:false,
								content:"该链接已失效",
								success() {
									that.$Router.replaceAll('/pages/index/index'); 
								}
							})
						}
					}
				});
			},
			openScan() {
				console.log('openScan123');
				// 只允许通过相机扫码
				let that = this;
				uni.scanCode({
					onlyFromCamera: true,
					scanType: ['qrCode'],
					success: function (res) {
						console.log('openScan_success');
						if(res.scanType == 'WX_CODE' && res.errMsg == 'scanCode:ok' && res.path){
							// 获取链接
							let temp = res.path;
							let temparr = temp.split('=');
							let goods_id = temparr[1];
							console.log('goods_id:'+goods_id);
							that.playvideo(goods_id);
						} else {
							uni.showModal({
								title:"错误!",
								showCancel:false,
								content:"二维码信息错误",
								success() {
									that.$Router.replaceAll('/pages/index/index'); 
								}
							})
						}
						
					},
					fail:function(err){
						console.log('openScan_err');
						that.$Router.replaceAll('/pages/index/index'); 
					}
				});
			},
			getBack(){
				const pages = getCurrentPages();  
				if (pages.length === 2) {
					uni.navigateBack({  
						delta: 1  
					});  
				} else if (pages.length === 1) {  
					this.$Router.replaceAll('/pages/index/index'); 
				} else {  
					uni.navigateBack({  
						delta: 1  
					});  
				} 	                
			}
		}
	}
</script>

<style>
	.scan{
		width: 100vw;
		height: 100vh;
	}
	video{
		width: 100%;
		height: 100%;
	}
</style>
