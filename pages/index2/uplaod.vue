<template>
  <view class="container">
    <text>上传成功</text>
  </view>
	<!-- <view class="containerr">
		<text class="close">时间： </text>
	</view>
	<view class="containerr">
		<text class="close">{{ currentTime }}</text>
	</view> -->
	<view class="containerrr">
		<image class="middle-image" src="/static/success.png"></image>
	</view>
	<view class="containerrr">
		<button class="button" @click="button">返回点检</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				 currentTime: ''
			}
		},
		methods: {
			updateTime() {
				const now = new Date();
				this.currentTime = `${now.getFullYear()} 年 ${now.getMonth() + 1} 月 ${now.getDate()} 日 ${now.getHours()} 时 ${now.getMinutes()} 分`;		
			console.log(this.currentTime)
			},
			button(){
				uni.navigateTo({
					url: '/pages/index2/index2'
				});
			}
		},
		 mounted() {
			this.updateTime(); // 初始化时间
			setInterval(() => {
				this.updateTime(); // 每60秒更新时间
			}, 60000);
		}
	}
</script>
 
<style>
	.header {
	  position: fixed;
	  top: 0;
	  left: 0;
	  right: 0;
	  height: 50px; /* 举例高度 */
	  background-color: #e7e6e6;
		font-size: 18px;
	  z-index: 10;
	
	}
	.text{
		display: flex;
		justify-content: center;
		align-items: center;
		height: 100%;
		color: #000000;
	}
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
	margin: 30px;
	margin-top: 50px;
	font-size: 32px;
}
.containerr{
	display: flex;
	justify-content: center;
	align-items: center;
}
.close{
	/* margin: 20px; */
	font-size: 16px;
}
  .containerrr {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100%; /* 使用视图的高度 */
		margin-top: 30px;
  }
 
  .middle-image {
    width: 150px; /* 图片宽度 */
    height: 150px; /* 图片高度 */
  }
	.button{
		background-color: azure;
		width: 350px;
	}
</style>