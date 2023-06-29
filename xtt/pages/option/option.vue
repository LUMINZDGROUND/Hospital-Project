<template>
	<view class="content">
		<view class="welcomeTitle">
			设置选项
		</view>
		
		<view class="iptView" v-for="(item,index) in titleData">
			<view class="title">{{item.title}}</view>
		    <input class="" focus v-model="configData[item.vmod]" :placeholder="item.placehold"/>
		</view>
		<button type="primary" @click="saveData">保存</button>
		<button type="warn" class="btnNext" @click="removeData">清除缓存</button>
		<button type="default" @click="backToLogin" class="btnNext backToLogin">返回</button>
	</view>
</template>

<script>
	export default {
		data(){
			return{
				configData:{
					ipAddress:'',
					// Url:'',
					portNumber:''
				},
				titleData:[
					{title:'ip地址',placehold:'请输入IP地址',vmod:'ipAddress'},
					// {title:'网址',placehold:'请输入网址',vmod:'Url'},
					{title:'端口号',placehold:'请输入端口号',vmod:'portNumber'},
				],
			}
		},
		onLoad: function (options) {
			// 页面初始化 options为页面跳转所带来的参数
			this.getConfig();
		},
		mounted() {
		  
		},
		methods:{
			getConfig() {
				uni.getStorage({
					key:'config',
						success:(res) => {
							this.configData = res.data,
							console.log(this.configData)
						}
				})
			},
			removeData() {
				uni.removeStorage({
					key:'config',
					success:function() {
						uni.showToast({
							title: '清除成功',
							duration: 2000,
						})
						// this.configData.ipAddress = ''
						// this.configData.portNumber = ''
					}
				})
			},
			saveData() {
				// uni.setStorageSync('config',this.configData)
				// uni.showToast({
				// 	title: '保存成功',
				// 	duration: 2000,
				// })
				// uni.navigateBack(1)
				uni.setStorage({
				key:'config',
				data:this.configData,
				success:function(){
					uni.showToast({
						title: '保存成功',
						duration: 2000,
						success:function() {
							uni.navigateBack({
								delta:1
							})
						}
					})
				}
			})
			},
			backToLogin() {
				uni.navigateBack(1)
				// uni.navigateTo({
				// 	url:'/pages/login/login'
				// })
			}
		},
	}
</script>

<style>
	page {
		background-color:#f8f8f8;
	}
	input {
		height: 96rpx;
		margin: 0 20rpx;
		line-height: 96rpx;
		text-align: center;
		border-radius: 30rpx;
		background-color: #fff;
		width: 96%;
		margin-bottom: 30rpx;
		box-shadow: 5rpx 5rpx 5rpx 5rpx #DCDCDC;
		display: flex;
		justify-content: center;
		align-items: center;
	}
	button {
		width: 720rpx;
		border-radius: 30rpx;
		margin-top: 100rpx;
	}
	.content {
		text-align: center;
		height: 1200rpx;
	}
	.title {
		font-size: 36rpx;
		margin-left: 32rpx;
		margin-bottom: 16rpx;
		text-align: left;
	},
	.welcomeTitle {
		height: 200rpx;
		line-height: 200rpx;
		font-size: 40rpx;
		text-align: center;
	}
	.iptView {
		text-align: center;
	}
	.btnNext {
		margin-top: 20rpx;
	}
	.backToLogin {
		border: 2rpx solid #D3D3D3;
	}
</style>