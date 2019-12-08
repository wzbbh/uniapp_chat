<template>
	<view class="content">
		<scroll-view id="scrollview" scroll-y="true" :scroll-top="scrollTop">
			<view class="list"  v-for="(obj,index) in list" :key="index">{{obj}}</view>
		</scroll-view>
		
		<!-- 底部 -->
		<view class="chatBox">
			<view class="inpBox"><input type="text"></view>
			<view class="sendBox" @click="send()"><text>发送</text></view>
		</view>
		<view class="sliderBox"></view>
	</view>
</template>
<script>
	var _this;
	export default {
		onLoad() {
			_this = this;
		},
	    data() {
	        return {
	            scrollTop: 0,				//设置竖向滚动条位置
				num		 : 0,
				list     : [],				//name名字，头像photo,时间time,内容cont,type(1是用户自己，2是对方)；
				
				sonHei   : 0,			    //所有子元素的高度
				winHei   : 0				//当前聊天窗口的高度
	        }
	    },
		mounted() {
			//第一步获取容器的可视化高度；
			const query = uni.createSelectorQuery().in(this);
			query.select('#scrollview').boundingClientRect(data => {
			    _this.winHei = data.height;
			}).exec();
		},
	    methods: {
			send(){
				this.list.push(this.num++);
				setTimeout(function(){
					_this.scrollToBottom();
				},300);
			},
			scrollToBottom(){				//滑动到底部
				_this.sonHei = 0;			//每次内部元素的高度先清空，因为下面会循环获取
				const query = uni.createSelectorQuery().in(this);
				query.selectAll('.list').boundingClientRect(data => {
				  data.forEach((res)=>{
					  _this.sonHei += res.height;  
				  });
				  const scrolly  =  _this.sonHei - _this.winHei;			//如果内部的元素高度 > 可视化高度，滑动的值就： 内部的元素高度-可视化高度；
				  if(_this.sonHei > _this.winHei){
				  	   _this.scrollTop = scrolly;
				  }
				}).exec();
			}
	    }
	}
</script>
<style>
	page{
		height: 100%;
	}
	.content{
		height: 100%;
	}
	#scrollview{
		height:calc(100% - 100rpx);
		border: 1rpx solid red;
	}
	
	.list{
		height: 100rpx;
		border: 1rpx solid red;
		box-sizing: border-box;
	}
	
	
	/* 底部样式 */
	.chatBox{
		width: 100%;
		height: 100rpx;
		padding: 0rpx 24rpx;
		border: 1rpx solid red;
		box-sizing: border-box;
		display: flex;
		align-items: center;
		justify-content: space-between;
		font-size: 32rpx;
	}
	.inpBox{
		flex: 1;
	}
	.inpBox input{
		height: 48rpx;
		background:#2C405A;
		color: white;
		border-radius:32rpx;
	}
	.sendBox{
		display: inline-block;
		margin-left: 24rpx;
	}
</style>