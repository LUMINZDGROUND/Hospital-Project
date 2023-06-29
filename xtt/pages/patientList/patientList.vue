<template>
  <view class="content">
    <view class="header">
      <button size="default" class="headBtn" v-for="(item, index) in headBtn" :key="index" :data-choose="index" @click="clickTab">{{ item }}</button>
    </view>
    <view class="patientContent">
      <button class="patientBtn bubble" v-for="(item, index) in chooseTab === 0 ? dqhz : ywhz" :key="index" :data-index="index" @click="goToXtQuery($event)">
        <view class="text_area">
			<text class="text_name">{{ item.brxm }}</text>
			<text class="text_sex_dah">性别:{{ item.xb }}&nbsp;&nbsp;&nbsp;&nbsp;档案号: {{ item.clinic_no }}</text>	
        </view>
	  </button>
    </view>
  </view>
</template>
<script>
export default {
  data() {
    return {
      headBtn: ['当前患者', '以往患者'],
      dqhz: [],
      ywhz: [],
	  chooseTab: 0, // 当前选中的选项卡：默认选中第一个~
	  selectDef:'',
	  configInfo:'',
	  daInfo:''
    };
  },
  mounted() {
	this.pdlogin();
    this.getAllPatient();
  },
  methods: {
	pdlogin() { // 登录判断，获取缓存中的loginInfo
		const loginInfo = uni.getStorageSync('loginInfo')
		if(loginInfo == '') {
			uni.navigateTo({
				url:'/pages/login/login'
			})
			uni.showToast({
				title: '您还没有登录，请先登录',
				icon: 'none',
				duration: 2000
			})  
		}
	},
	goToXtQuery(event) {
		const idx = event.target.dataset.index //获取点击按钮的索引
		// console.log(idx)
		let codeInfo;
		if(this.chooseTab === 0) {
			codeInfo = this.dqhz[idx]  // 获取当前患者点击按钮的住院号、档案号、病历号
			uni.setStorageSync('codeInfo',codeInfo)
		} else {
			codeInfo = this.ywhz[idx] // 获取以往患者点击按钮的住院号、档案号、病历号
			uni.setStorageSync('codeInfo',codeInfo)
		}
		uni.navigateTo({
			url:'/pages/xtQuery/xtQuery'
		})
	},
     getAllPatient() {
		try{
			let that = this
			 uni.getStorage({
				key:"config",
				success:(res) => {
					// console.log('本地缓存ip:'+ res.data.ipAddress)
					// console.log('本地缓存端口号:'+ res.data.portNumber)
					that.configInfo = 'http://' + res.data.ipAddress + ':' + res.data.portNumber + '/Emergency/getAllXtinfo'
					// console.log(that.configInfo)
					uni.request({
						url:that.configInfo,
						success: (res) => {
						const csml = res.data.data;
						console.log('csml:',csml);
						let config = uni.getStorageSync("config");
						for (let i = 0; i < csml.length; i++) {
							const dah = csml[i].clinic_no
							uni.request({ // 获取姓名、性别
								url:'http://' + config.ipAddress + ':' + config.portNumber + '/Emergency/getNameByDah',
								method:"POST",
								data: {
									dah: dah
								},
								success:(rbs) => {
									if (csml[i].jlzt === '1') { // 如果记录状态为1,将csml[i]连同对应的姓名性别添加到当前患者
									this.dqhz.push(Object.assign({}, csml[i], { brxm:rbs.data.data[0].brxm, xb:rbs.data.data[0].xb }));
									} else { 					// 如果记录状态为2,将csml[i]连同对应的姓名性别添加到以往患者
									this.ywhz.push(Object.assign({}, csml[i], { brxm:rbs.data.data[0].brxm, xb:rbs.data.data[0].xb }));
									}
								}
							})
						}
						// console.log('dqhz:', this.dqhz);
						// console.log('ywhz:', this.ywhz);
						},
					});
				}
			})
		}catch(error){
			console.log(error)
		}
		
		
	},
	clickTab: function(e) {
	    this.chooseTab = parseInt(e.currentTarget.dataset.choose);
	},
  },
};
</script>


<style>
	page {
		background-color: #f8f8f8;
	}
	button::after{ 
		border: none;
	}
	text {
		margin: 0;
	}
	.header {
		text-align: center;
	}
	.text_area {
		position: absolute;
		top: 24rpx;
	}
	.text_sex_dah {
		display: block;
		font-size: 25rpx;
		color: #808080;
	}
	.headBtn {
		outline: none;
		border: none;
		display: inline-block;
		margin: 15rpx;
		width: 330rpx;
		background-color: #fff;
		/* border-radius: 35rpx; */
		/* box-shadow: 5rpx 5rpx 5rpx 5rpx RGB(245,245,245); */
	}
	.patientBtn {
		position: relative;
		outline: none;
		border: none;
		width: 695rpx;
		height: 120rpx;
		/* border-radius: 40rpx; */
		line-height: 120%;
		text-align: left;
		padding-left: 10%;
		font-size: 32rpx;
		font-weight: 600;
		margin-bottom: 2rpx;
		background-color: #fff;
		/* box-shadow: 5rpx 5rpx 5rpx 5rpx RGB(241,241,241); */
	}
	.bubble {
		position: relative;
		overflow: hidden;
	}
	.bubble:after {
		content: "44444";
		background: #5c9899;
		position: absolute;
		width: 750rpx;
		height: 750rpx;
		left: calc(50% - 375rpx);
		top: calc(50% - 375rpx);
		opacity: 0;
		margin: auto;
		border-radius: 50%;
		transform: scale(1);
		transition: all 0.5s ease-in-out;
	}
</style>