<template>
	<view class="content">
		<view class="header">
			姓名:{{ brxm }}&nbsp; 性别:{{ xb }}&nbsp; 档案号:{{ clinic_no }}
		</view>
		<scroll-view scroll-y="true" class="choose-tab" :style="'height:'+(this.h-110)+'px'">
			<view class="choose-item" v-for="(item,index) in leftTabList" :class="chooseTab == index ? 'active' : ''" :data-choose="index" @click="clickTab">
				{{ item.name }}
			</view>
		</scroll-view>
		<scroll-view scroll-y="true" class="rightContent" :style="'height:'+(this.h-110)+'px'">
			<!-- {{fromData}} -->
		    <view v-for="(dt,i) in leftTabList[chooseTab].content">
		      <view v-if="dt.type === 'dateTime'">
		        <view class="title">{{ dt.inputLabel }}</view>
		        <view class="example-body">
		          <uni-datetime-picker type="datetime" v-model="fromData[dt.modelName]" @change="changeLog" />
		        </view>
		      </view>
		      <view v-else-if="dt.type === 'checkbox'">
		        <view class="title">{{ dt.inputLabel }}</view>
		        <checkbox-group @change="checkboxChange" v-model="fromData[dt.modelName]" :data-modelName="dt.modelName">
		          <label v-for="(et,ide) in dt.data" >
		            <checkbox :checked="fromData[dt.modelName] && fromData[dt.modelName].includes(et.value)" :value="et.value" />{{ et.label }}
		          </label>
		        </checkbox-group>
		      </view>
		      <view v-else-if="dt.type === 'input'">
		        <view class="title">{{ dt.inputLabel }}</view>
		        <input class="uni-input" v-model="fromData[dt.modelName]" :placeholder="dt.placeHolder" /> <span>{{dt.unit}}</span>
		      </view>
		      <view v-else-if="dt.type === 'radio'" >
		        <view class="title">{{ dt.inputLabel }}</view>
		        <radio-group @change="radioChange" v-model="fromData[dt.modelName]" :data-modelName="dt.modelName">
		          <label v-for="(et,idx) in dt.data">
		            <radio :checked="fromData[dt.modelName] && fromData[dt.modelName].includes(et.value)" :value="et.value" />{{ et.label }}
		          </label>
		        </radio-group>
		      </view>
		    </view>
		</scroll-view>
		<button type="warn" @click="clearData">清空</button>
		<button type="primary" @click="submit">提交</button>
	</view>
</template>

<script>
	export default {
		data(){
			return{
				// ----------------------------------------
				zyh:'',
				clinic_no:'',
				clinic_regno:'',
				brxm:'',
				xb:'',
				// ----------------------------------------
				netConfig:'',
				// ----------------------------------------
				leftTabList:[{
					zyh:'',
					clinic_no:'',
					clinic_regno:''
				}],
				chooseTab: 0, // 当前选中的选项卡：默认选中第一个~
				fromData:{
				},
				
			}
		},
		onLoad: function (options) {
			// 页面初始化 options为页面跳转所带来的参数
			this.getCodeInfo();
			this.getLeftTab();
			// if(fromData!=null) {
			// 	this.getAllXtinfoByZCC();
			// }
			this.getAllXtinfoByZCC();
			// this.codeInfo = JSON.parse(decodeURIComponent(options.codeInfo));
		},
		mounted() {
		  this.fromData.zyh = this.zyh;
		  this.fromData.clinic_no = this.clinic_no;
		  this.fromData.clinic_regno = this.clinic_regno;
		  this.h=uni.getSystemInfoSync().windowHeight;
		  // +uni.upx2px(1020);
		},
		methods:{
			getLeftTab() {
				const system = uni.getSystemInfoSync().windowHeight;
				console.log(system);
				uni.request({
					url:"http://" + this.netConfig.ipAddress + ":" + this.netConfig.portNumber + "/Emergency/getAllEmergency",
					method:'GET',
					success:(res) => {
						console.log("res.data.data:" + res.data.data)
						// if (res.data && res.data.data && res.data.data.content) {
							this.leftTabList = res.data.data
							console.log(this.leftTabList)
						// } else {
						// 	console.log(this.leftTabList)
						// 	uni.showToast({
						// 		title: '错误，没有获取到数据！',
						// 		icon: 'none',
						// 		duration: 2000
						// 	}) 
						// 	// 处理数据为空的情况
						// }
						// console.log(res)
						// this.leftTabList = res.data.data.map(item => {
						// return { 
						// 	...item,
						// 	content:JSON.parse(item.content).replace(/[\\]/g,'')
						// 		};
						// });
						// console.log(this.leftTabList)
					},
					fail:(err)=>{
						console.log(err)
					}
				})
			},
			async getAllXtinfoByZCC() {
				uni.request({
					url:"http://" + this.netConfig.ipAddress + ":" + this.netConfig.portNumber + "/Emergency/getAllXtinfoByZCC",
					method:'GET',
					data:{
						zyh:this.zyh,
						clinic_no:this.clinic_no,
						clinic_regno:this.clinic_regno
					},
					success:(res)=>{
						console.log("getAllXtinfoByZCC:" + res.data)
						if(res.data.data[0]!= null) { // 根据三个字段获取的数据不为空，查到（存在）
							this.fromData = res.data.data[0]
							console.log(this.fromData)
						} else  { //查不到，不存在
							
						}
					},
				})
			},
			getCodeInfo() { // 获取点击按钮缓存中的姓名、性别、档案号、住院号、流水号
				const cc = uni.getStorageSync('codeInfo')
				this.zyh = cc.zyh;
				this.clinic_no = cc.clinic_no;
				this.clinic_regno = cc.clinic_regno;
				this.brxm = cc.brxm;
				this.xb = cc.xb;
				this.netConfig = uni.getStorageSync("config")
			},
			radioChange: function(e) {
				this.fromData={
					...this.fromData,
					[e.mp.target.dataset.modelname]:e.detail.value
				}
			},
			checkboxChange: function (e) {
				this.fromData={
					...this.fromData,
					[e.mp.target.dataset.modelname]:e.detail.value
				}
			},
			submit() { // 提交按钮事件
				const data = {
					zyh:this.zyh,
					clinic_no:this.clinic_no,
					clinic_regno:this.clinic_regno,
					...this.fromData
				}
				uni.showModal({// 弹出信息确认对话框
					title:'提示',
					content:'你确定要提交吗？',
					success:(res) => {
						if(res.confirm) { //用户点击确定
							//------------------------------------------------------------------------------------------------------------------------------
							/**
							* uni.checkIsSupportSoterAuthentication:  获取本机支持认证方式（ 
							*         res.supportMode = ['fingerPrint'] 只支持指纹识别
							*         res.supportMode = [] 不具备任何被SOTER支持的生物识别方式
							*         res.supportMode = ['fingerPrint', 'facial'] 支持指纹识别和面容识别
							* ）
							* 需求：当前业务只要求指纹识别功能
							* 流程：获取本机支持的生物认证方式，获取设备内是否有生物信息，进行生物认证
							* 逻辑：
							* ①先获取设备支持的生物认证方式:如果获取失败，则弹出"此设备不支持指纹/面容识别"提示框；如果成功，则执行第②步
							* ②判断当前设备生物识别中是否包括指纹或面容识别:如果不包括，则弹出"此设备不支持指纹/面容识别功能"；如果包括，则执行第③步
							* ③获取设备内是否有生物认证信息;认证方式：指纹/面容，如果没有，弹出"此设备未录入指纹，请设置"的提示框；如果有，则执行第④步
							* ④判断第③步获取成功的信息：如果res.isEnrolled为假，则弹出"此设备未录入指纹，请到设置中开启"提示框；如果res.isEnrolled为真，则执行第⑤步
							* ⑤开始进行生物认证，输入认证方式、挑战因子、验证描述信息：如果验证失败，则弹出"验证失败"提示框；如果验证成功，执行提交命令
							* ⑥最后接口调用结束，执行complete
							*/
							uni.checkIsSupportSoterAuthentication({ // 获取本机支持的生物认证方式
								success:(res) => { // 获取本机支持的认证方式成功
									console.log('supportModeSuccess:' + res)
									if(res.supportMode && res.supportMode.includes('fingerPrint')) { // 如果当前设备支持生物识别方式，且支持指纹识别方式
										uni.checkIsSoterEnrolledInDevice({ // 获取设备内是否有生物信息
											checkAuthMode:'fingerPrint' ,// 认证方式:检验指纹信息
											success:(res) => { // 成功，有指纹信息
												console.log(res.isEnrolled)
												if(res.isEnrolled == true) { //如果为真
													uni.startSoterAuthentication({ //开始生物认证
														requestAuthModes:['fingerPrint'],// 可接受的生物认证方式:fingerPrint(指纹)、facial(面容)
														challenge:'123456', // 挑战因子:此次生物鉴权准备的用于签名的字符串关键识别信息，将作为 resultJSON 的一部分，供调用者识别本次请求
														authContent:'请验证指纹', // 验证描述信息
														success:(res) => { // 验证成功后的操作
															console.log(res)
															uni.showToast({
																title: "验证成功",
																duration: 2000,
																icon:'none'
															})
															// 提交操作 -----------------------------------------------------------------------------------------------------
															for(let i = 0 ; i < this.leftTabList.length ; i++ ) {
																for(let j = 0 ; j < this.leftTabList[i].content.length ; j++) {
																	if(this.leftTabList[i].content[j].type === 'checkbox' && this.leftTabList[i].content[j].modelName in data) {
																		// 判断包含data是否为空
																		if (data[this.leftTabList[i].content[j].modelName] && Array.isArray(data[this.leftTabList[i].content[j].modelName]) && data[this.leftTabList[i].content[j].modelName].length === 0) {
																		    // 如果为空：不包含，'',
																			data[this.leftTabList[i].content[j].modelName] = '';
																		} else if (Array.isArray(data[this.leftTabList[i].content[j].modelName])) {
																		    data[this.leftTabList[i].content[j].modelName] = data[this.leftTabList[i].content[j].modelName].join(',');
																		}
																	}
																}
															}
															uni.request({
																url:"http://" + this.netConfig.ipAddress + ":" + this.netConfig.portNumber + "/Emergency/getAllXtinfoByZCC",
																method:'GET',
																data:{
																	zyh:this.zyh,
																	clinic_no:this.clinic_no,
																	clinic_regno:this.clinic_regno
																},
																success:(res)=>{
																	if(res.data.data[0]!= null) { // 查到，不为空（存在）
																		uni.request(
																		{ // 修改已有数据，并存入数据库
																			url:"http://" + this.netConfig.ipAddress + ":" + this.netConfig.portNumber + "/Emergency/xtUpdate",
																			method:'POST',
																			data:data
																		})
																	} else { // 查不到，不存在
																		uni.request({ // 提交新数据，存入数据库
																			url:"http://" + this.netConfig.ipAddress + ":" + this.netConfig.portNumber + "/Emergency/submit",
																			method:'POST',
																			data
																		})
																	}
																},
															})
															uni.showToast({ // 弹出提示框
																title: '提交成功',
																icon: 'success', // 将值设置为 success 或者直接不用写icon这个参数
																duration: 2000 // 显示持续时间为 2秒
															})  
															// 提交操作 -------------------------------------------------------------------------------------------------------
														},
														fail: (err) => {  // 验证失败后的操作
															console.log('error:'+ err +'6666666666666')
															uni.showToast({
																title:"验证失败",
																duration:2000,
																icon:'none'
															})
														},
														complete: (res) => { // 接口调用结束，调用成功或失败都会执行
															console.log('complete:' + res)
														}
													})
												} else { // 如果为假
													uni.showToast({
														title:'此设备未录入指纹，请到设置中开启',
														duration:4000,
														icon:'none'
													})
												}
											},
											fail: (err) => { // 失败，无指纹信息
												console.log('失败，无指纹信息')
												uni.showToast({
													title:'此设备未录入指纹，请设置',
													duration:2000,
													icon:'none'
												})
											}
										})
									} else { // 如果当前设备不支持支持生物识别方式
										console.log("此设备不支持指纹识别功能")
										uni.showToast({
											title:'此设备不支持指纹识别功能',
											duration:2000,
											icon:'none'
										})
									}
								},
								fail: (err) => { // 获取本机支持的认证方式失败
									uni.showToast({
										title:'此设备不支持指纹识别功能',
										duration:2000,
										icon:'none'
									})
								}
							})
							//------------------------------------------------------------------------------------------------------------------------------
						} else if(res.cancel) { // 用户点击取消
							
						}
					}
				})
			},
			clearData() {
				uni.showModal({
					title:'提示',
					content:'你确定要清空所填吗？此操作不可逆',
					success:(res) => {
						if(res.confirm) { // 用户点击确定
							this.fromData = '',
							this.fromData = {
								zyh:this.zyh,
								clinic_no:this.clinic_no,
								clinic_regno:this.clinic_regno,
							},
							uni.request({ // 提交新数据，存入数据库
								url:"http://"+ this.netConfig.ipAddress + ":" + this.netConfig.portNumber +"/Emergency/xtUpdate",
								method:'POST',
								data:this.fromData,
							})
							uni.showToast({ //弹出提示框
								title: '已清除',
								icon: 'success', //将值设置为 success 或者直接不用写icon这个参数
								duration: 2000 //显示持续时间为 2秒
							})  
						} else if(res.cancel) { //用户点击取消
							
						}
					},
				})
				
			},
			clickTab: function(e) {
			    this.chooseTab = parseInt(e.currentTarget.dataset.choose);
			    // console.log(this.chooseTab);
			},

			changeLog(value) {
				// 处理日期时间选择器的变化事件
				console.log('Selected datetime:', value);
			}
		},
	}
</script>

<style>
	button::after{
		border: none;
		/* position: fixed;
		right: 20px;
		bottom: calc( var(--window-bottom) + 20px);
		z-index: 1030;
		margin-bottom: 6; */
	}
	label {
		display: inline-block;
		width: 600rpx;
		height: 65rpx;
	}
	
	input {
		display: inline-block;
		width: 400rpx;
		height: 70rpx;
		font-size: 18px;
		border-bottom: 1px solid gray;
	}
	button {
		float: left;
		width: 350rpx;
		margin-left: 20rpx;
	}
	.example-body {
		width: 490rpx;
	}
	.content {
	}
	.header {
		height: 80rpx;
		line-height: 80rpx;
		font-size: 17px;
		font-weight: 700;
		text-align: center;
		background-color: #f8f8f8;
	}
	.title {
		margin-top: 30rpx;
		margin-bottom: 15rpx;
	}
	.boldTitle {
		font-size: 18px;
		font-weight: 700;
	}
	.choose-tab {
		float: left;
		display: inline-block;
		width: 220rpx;
		height: 1320rpx;
		color: #778899;
		line-height: 90rpx;
		font-weight: 700;
		
	}
	
	.choose-item {
		background-color: #F7F7F7;
		text-align: right;	
		padding-right: 20rpx;
	}
	
	.active {
		background-color: white;
	}
	.rightContent {
		display: inline-block;
		float: left;
		width: 490rpx;
		height: 1300rpx;
		/* margin-left: 25%; */
		padding: 15rpx;
	}
	
	.rightTab {
		float: left;
		width: 545rpx;
		height: 1050rpx;
	}
	.datetime{
		color: #CCCCCC;
	}
</style>