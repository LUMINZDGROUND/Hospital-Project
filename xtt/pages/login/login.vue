<template>
	<view class="content">
		<view class="welcomeTitle">
			欢迎使用
		</view>
		<view class="iptView">
		    <input v-model="userInfo.bm" placeholder="请输入账号" />
		</view>
		<view class="iptView">
		    <input type="password" v-model="userInfo.mm" placeholder="请输入密码" />
		</view>
		<button type="primary" @click="login">登录</button>
		<button type="warn" @click="clear">清空</button>
		<button type="default" class="netOption" @click="netOption"><uni-icons type="gear-filled" size="25"/></button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				userInfo:{
					bm:'',
					mm:''
				},
				configInfo:''
			}
		},
		onLoad: function (options) {
			// 页面初始化 options为页面跳转所带来的参数(运行方法方法)
			this.readConfigFile();
		},
		mounted() {
		},
		methods:{
			readConfigFile() { // 从缓存中获取config配置信息
				// this.configInfo = "http://" + cc.ipAddress +':'+ cc.portNumber + '/wx/mini/getAllUser' // 获取输入的网址
				const cc = uni.getStorageSync("config");
				console.log(cc)
				if(cc === '') {
					uni.navigateTo({
						url:"/pages/option/option"
					}),
					uni.showToast({
						title: '您还没有设置网络选项，请先设置',
						icon: 'none',
						duration: 2000
					})
				} else {
					this.configInfo = "http://" + cc.ipAddress + ':' + cc.portNumber + '/Emergency/getAllUser' // 获取输入的网址
				}
				uni.removeStorageSync('loginInfo')
				// uni.getStorage({ 
				// 	key:"config",
				// 	success:(res) => {
				// 		console.log(res.data)
				// 		this.configInfo = "http://" + res.data.ipAddress + ':' + res.data.portNumber + '/Emergency/getAllUser' // 获取输入的网址
				// 	},
				// 	fail:(res) => {
				// 		uni.navigateTo({
				// 			url:"/pages/option/option"
				// 		}),
				// 		uni.showToast({
				// 			title: '您还没有设置网络选项，请先设置',
				// 			icon: 'none',
				// 			duration: 2000
				// 		})  
				// 	}
				// })
			},
			login() { // 登录
				if(this.userInfo.bm == '' && this.userInfo.mm == '') {
					uni.showToast({
						title: '账号和密码为空，请输入账号密码',
						icon: 'none',
						duration: 2000
					})
				} else if(this.userInfo.bm == '') {
					uni.showToast({
						title: '账号为空，请输入账号',
						icon: 'none',
						duration: 2000
					})
				} else if (this.userInfo.mm == '') {
					uni.showToast({
						title: '密码为空，请输入账号',
						icon: 'none',
						duration: 2000
					})
				} else {
					let that = this
					const cc = uni.getStorageSync("config");
					const configInfo = "http://" + cc.ipAddress + ':' + cc.portNumber + '/Emergency/getAllUser' // 获取输入的网址
					console.log("that.configInfo:" + that.configInfo)
					  const data = {
					    ...that.userInfo
					  };
					  // console.log("账号：" + data.bm,"密码：" + data.mm)
					  uni.request({ // 输入账号后发送请求
						url:configInfo, // 获取到从缓存中的从设置选项输入的网址
					    method: "POST",
						sslVerify:false,
					    data,
					    success:(res) => {
							uni.setStorageSync('loginInfo','已登录') // 登录判断：向缓存中存入token
							if(res.data.code == 200) {
								// 生物识别
								// //------------------------------------------------------------------------------------------------------------------------------
								// /**
								// * uni.checkIsSupportSoterAuthentication:  获取本机支持认证方式（ 
								// *         res.supportMode = ['fingerPrint'] 只支持指纹识别
								// *         res.supportMode = [] 不具备任何被SOTER支持的生物识别方式
								// *         res.supportMode = ['fingerPrint', 'facial'] 支持指纹识别和面容识别
								// * ）
								// * 需求：当前业务只要求指纹识别功能
								// * 流程：获取本机支持的生物认证方式，获取设备内是否有生物信息，进行生物认证
								// * 逻辑：
								// * ①先获取设备支持的生物认证方式:如果获取失败，则弹出"此设备不支持指纹/面容识别"提示框；如果成功，则执行第②步
								// * ②判断当前设备生物识别中是否包括指纹或面容识别:如果不包括，则弹出"此设备不支持指纹/面容识别功能"；如果包括，则执行第③步
								// * ③获取设备内是否有生物认证信息;认证方式：指纹/面容，如果没有，弹出"此设备未录入指纹，请设置"的提示框；如果有，则执行第④步
								// * ④判断第③步获取成功的信息：如果res.isEnrolled为假，则弹出"此设备未录入指纹，请到设置中开启"提示框；如果res.isEnrolled为真，则执行第⑤步
								// * ⑤开始进行生物认证，输入认证方式、挑战因子、验证描述信息：如果验证失败，则弹出"验证失败"提示框；如果验证成功，则跳转至目标页面
								// * ⑥最后接口调用结束，执行complete
								// */
								// uni.checkIsSupportSoterAuthentication({ // 获取本机支持的生物认证方式
								// 	success:(res) => { // 获取本机支持的认证方式成功
								// 		console.log('supportModeSuccess:' + res)
								// 		if(res.supportMode && res.supportMode.includes('fingerPrint')) { // 如果当前设备支持生物识别方式，且支持指纹识别方式
								// 			uni.checkIsSoterEnrolledInDevice({ // 获取设备内是否有生物信息
								// 				checkAuthMode:'fingerPrint' ,// 认证方式:检验指纹信息
								// 				success:(res) => { // 成功，有指纹信息
								// 					console.log(res.isEnrolled)
								// 					if(res.isEnrolled == true) { //如果为真
								// 						uni.startSoterAuthentication({ //开始生物认证
								// 							requestAuthModes:['fingerPrint'],// 可接受的生物认证方式:fingerPrint(指纹)、facial(面容)
								// 							challenge:'123456', // 挑战因子:此次生物鉴权准备的用于签名的字符串关键识别信息，将作为 resultJSON 的一部分，供调用者识别本次请求
								// 							authContent:'请验证指纹', // 验证描述信息
								// 							success:(res) => { // 验证成功后的操作
								// 								console.log(res)
								// 								uni.showToast({
								// 									title: "验证成功",
								// 									duration: 2000,
								// 									icon:'none'
								// 								})
								// 								console.log('登录成功!')
								// 								uni.navigateTo({
								// 									url:"/pages/patientList/patientList"
								// 								})
								// 							},
								// 							fail: (err) => {  // 验证失败后的操作
								// 								console.log('error:'+err+'6666666666666')
								// 								uni.showToast({
								// 									title:"验证失败",
								// 									duration:2000,
								// 									icon:'none'
								// 								})
								// 							},
								// 							complete: (res) => { // 接口调用结束，调用成功或失败都会执行
								// 								console.log('complete:' + res)
								// 							}
								// 						})
								// 					} else { // 如果为假
								// 						uni.showToast({
								// 							title:'此设备未录入指纹，请到设置中开启',
								// 							duration:4000,
								// 							icon:'none'
								// 						})
								// 					}
								// 				},
								// 				fail: (err) => { // 失败，无指纹信息
								// 					console.log('失败，无指纹信息')
								// 					uni.showToast({
								// 						title:'此设备未录入指纹，请设置',
								// 						duration:2000,
								// 						icon:'none'
								// 					})
								// 				}
								// 			})
								// 		} else { // 如果当前设备不支持支持生物识别方式
								// 			console.log("此设备不支持指纹识别功能")
								// 			uni.showToast({
								// 				title:'此设备不支持指纹识别功能',
								// 				duration:2000,
								// 				icon:'none'
								// 			})
								// 		}
								// 	},
								// 	fail: (err) => { // 获取本机支持的认证方式失败
								// 		uni.showToast({
								// 			title:'此设备不支持指纹识别功能',
								// 			duration:2000,
								// 			icon:'none'
								// 		})
								// 	}
								// })
								// //------------------------------------------------------------------------------------------------------------------------------
								uni.showToast({
									title: '登录成功',
									icon: 'none',
									duration: 2000
								})
								console.log('登录成功!')
								uni.navigateTo({
									url:"/pages/patientList/patientList"
								})
							} else {
								uni.showToast({
									title: '账号或密码错误',
									icon: 'none',
									duration: 2000
								})  
							}
					    },
						fail: (res) => {
							console.log(res)
							uni.showToast({
								title: '连接失败，请检查ip地址或端口号是否正确',
								icon: 'none',
								duration: 2000
							})
						}
					  });
				}
			},
			FingerPrintRecognition() {
				
			},
			clear() {
				this.userInfo.bm = '',
				this.userInfo.mm = ''
			},
			netOption() {
				uni.navigateTo({
					url:"/pages/option/option"
				})
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
		margin-top: 20rpx;
	}
	button::after{
		border: none;
	}
	.content {
		position: relative;
		text-align: center;
		height: 1200rpx;
		line-height: 500rpx;
	}
	.welcomeTitle {
		font-size: 40rpx;
		text-align: center;
	}
	.iptView {
		text-align: center;
	}
	.netOption {
		position: absolute;
		bottom: 20rpx;
		right: 20rpx;
		outline: none;
		border: none;
		width: 100rpx;
		border: none;
		height: 100rpx;
		text-align: center;
		line-height: 100rpx;
	}
</style>