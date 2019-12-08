<template>
	<view class="content">
		<scroll-view id="scrollview"  scroll-y="true" :scroll-top="scrollTop"  :style="{height:winHei+'px'}"     @touchstart="touchStart"  >
			<view class="list"  v-for="(obj,index) in list" :key="index">
				<!-- 时间 -->
				<view class="chatTime">{{obj.time}}</view>
				<!-- 内容-用户的内容服务器端 -->
				<view class="chatInfo">
					<image src="../../static/logo.png" mode=""></image>
					<view>
						<view class="sjx"></view>
						<rich-text :nodes="obj.name.replace(/\#[\u4E00-\u9FA5]{1,3}\;/gi, emotion)   "></rich-text>
					</view>
				</view>
				<!-- 留点空隙代替padding-bottom -->
				<view class="jx" v-if="index==list.length-1"></view>
			</view>
		</scroll-view>
		
		<!-- 底部 -->
		<view class="bottom">
			<view class="chatBox"> 
				<view class="Emjo"    @tap="chooseEmjo">表情</view>
				<view class="inpBox"><input  id="inp" type="text"  @focus="focus"  @input="input" v-model="inpText"></view>
				<view class="sendBox" @click="send()"><text>发送</text></view>
			</view>
			<!-- 表情容器 -->
			<view class="sliderBox" :class="{'sliderBoxAc':isShow}">
				<swiper class="swiper">
					<swiper-item class="swiperItem" v-for="(a,index) in emoji.page" :key="index">
						<view  v-for="(b,index2) in emoji.faceNum" :key="index2">
							<!-- {{index2+emoji.faceNum*index}}  index=0, 0-24,index=1, 24-48 -->
							<image :src="emoji.FaceList[index2+emoji.faceNum*index].arry"  @click="choose_face(index2+emoji.faceNum*index)" mode="widthFix"></image>
						</view>
					</swiper-item>
				</swiper>
			</view>
		</view>
		
		<!-- 这个nothing其实就是sliderBox的高度,js动态绑定需要该高度 -->
		<view class="nothing"></view>		
	</view>
</template>
<script>
	var _this;
	var util = require('../../common/emojis.js');
	export default {
		onLoad() {
			_this = this;
			_this.getFaceList();
		},
	    data() {
	        return {
	            scrollTop: 0,				//设置竖向滚动条位置
				list     : [],				//name名字，头像photo,时间time,内容cont,type(1是用户自己，2是对方)；
				sonHei   : 0,			    //所有聊天内部子元素的高度
				winHei   : 0,				//当前聊天窗口容器的高度
				isShow   : false,			//是否显示表情
				inpText  : "",				//输入框内容
				emojiHtml: "",				//展示输入框内容转换后的
				
				emoji    : {
					 page     : 0,				//总共有几页
					 faceNum  : 24,				//每页显示24个图标
					 FaceList : []				//总表情数
				},
				replaceArry : []
	        }
	    },
		mounted() {
			_this.scrollToBottom();			
		},
	    methods: {
			input(event){
				console.log(JSON.stringify(event.detail.value));
			},
			focus(event){						//聚焦时候页面往上推，自动滑动到底部；
				_this.scrollTop = 1000*_this.list.length;	
				_this.isShow = false;
				
				var  my = window.getSelection().getRangeAt(0);
				console.log(": " + JSON.stringify(my));
				// console.log(JSON.stringify(event));
			},
			touchStart(){				    //软键盘失去焦点不用再判断了，pagejson里面已经配置了，非input区域会关闭软键盘，这里是input没有聚焦的时候，点击其它地方关闭表情并滑动到底部
				_this.isShow = false;
				setTimeout(function(){	
					_this.scrollToBottom();
				},10);
			},
			chooseEmjo(){				    //表情选择,防止有软键盘的时候先关闭软键盘显示会友好点
				setTimeout(function(){
					_this.isShow = !_this.isShow;
					if(_this.isShow){
						_this.scrollToBottom();
					}else{
						setTimeout(function(){
							_this.scrollToBottom();
						},150)
					}
				},100)
			},
			send(){						    //发送
				_this.setData();
			},
			setData(){						//设置发送的数据(模拟)
				if(_this.inpText.trim() == ''){
					uni.showToast({
						title : "不能发表空消息"
					});
					return
				}
				// {1-5}中间中文可以最多5位  //
				_this.emojiHtml = _this.inpText.replace(/\#[\u4E00-\u9FA5]{1,5}[0-9]*\;/gi,_this.emotion);		
				var obj = {
					name: _this.emojiHtml,
					time: _this.setTime()
				}
				_this.list.push(obj);
				setTimeout(function(){
					_this.inpText = "";			//发表结束清空输入框内容
					_this.scrollToBottom();
				},100);
			},
			scrollToBottom(){				//滑动到底部方法，每次动态计算聊天窗口高度；
				_this.sonHei= 0;			//每次内部元素的高度先清空，因为下面会循环获取
				const query = uni.createSelectorQuery().in(this);
				query.select('.chatBox').boundingClientRect();
				query.select('.content').boundingClientRect();
				query.select('.nothing').boundingClientRect();
				query.selectAll('.list').boundingClientRect();
				query.exec((res)=>{
					res[3].forEach((resSon)=>{
						_this.sonHei += resSon.height;  
					});
					if(_this.isShow){
						_this.winHei  = res[1].height - res[0].height -res[2].height;
					}else{
						_this.winHei  = res[1].height - res[0].height;
					}
					setTimeout(function() {
						const scrolly  =  _this.sonHei - _this.winHei;			//如果内部的元素高度 > 可视化高度，滑动的值就： 内部的元素高度-可视化高度；
						if(_this.sonHei > _this.winHei){
							_this.scrollTop = scrolly;
						}
					},10);
				})
			},
			choose_face(cont){
				_this.inpText = _this.inpText + _this.emoji.FaceList[cont].emoji;
			},
			getFaceList(){
				for(var i=0;i<= util.emojiArry.length;i++){
					var src={
						arry : ".../../static/arclist/"+i+".gif",
						emoji: "#"+util.emojiArry[i]+";"
					}
					_this.emoji.FaceList.push(src);
				}
				_this.emoji.page = Math.ceil(_this.emoji.FaceList.length/_this.emoji.faceNum);	//不如2.5页就显示3页
			},
			setTime(){			//每次发送的当前时间;
				var data= new Date();
				var hour= data.getHours();
				var min = data.getMinutes();
				if(hour>12){
					hour = "下午 " + hour;
				}else{
					hour = "上午 " + hour;
				}
				if(min<10){
					min = "0"+min;
				}
				return hour+":"+min;
			},
			emotion(res){
				let txt = res.replace(/\#|\;/gi,'');						//去掉#跟;再去数组中去匹配
				let index = util.emojiArry.indexOf(txt);
				return `<img src='.../../static/arclist/${index}.gif' />`
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
	
	
	
	
	/* 服务端用户内容 */
	.chatTime{
		height:100rpx;
		line-height:100rpx;
		text-align: center;
		font-size: 28rpx;
		color: #C8C7CC;
	}
	.chatInfo{
		position: relative;
		padding: 0rpx 24rpx;
		box-sizing: border-box;
		display: flex;
		font-size: 32rpx;
	}
	.chatInfo>image{
		width:88rpx;
		height:88rpx;
	}
	.chatInfo>view{
		max-width:70%;
		display: flex;
		align-items: center;		
		text-align: justify;
		padding:12rpx 24rpx;
		border-radius: 8rpx;
		box-sizing: border-box;
		background: #2C405A;
		color: white;
		margin-left: 24rpx;
		position: relative;
	}
	.emojiImg{
		vertical-align: middle;
	}
	.sjx{		/* 小三角形 */
		width: 20rpx;
		height: 20rpx;
		background: #2C405A;
		position: absolute;
		top: 34rpx;
		left: -6rpx;
		transform: rotate(45deg);
		transform-origin: center center;
	}
	.jx{       /*聊天窗口底部间距*/
		width: 100%;
		height: 40rpx;
	}
	/* 服务端用户内容结束 */
	
	
	
	
	
	
	/* 底部样式 */
	.bottom{
		width: 100%;
		box-sizing: border-box;
		position: fixed;
		bottom: 0rpx;
		left: 0rpx;
	}
	.chatBox{
		width: 100%;
		height: 100rpx;
		box-sizing: border-box;
		display:flex  !important; 
		align-items: center;
		font-size: 32rpx;
		background:#2C405A;
		color:white;
	}
	.Emjo{
		height: 100rpx;
		line-height: 100rpx;
		padding:0rpx 24rpx;
	}
	.inpBox{		/*输入框容器*/
		flex: 1;
	}
	.inpBox input{
		height:60rpx !important;
		background:white;
		color: white;
		border-radius:32rpx;
		padding:0rpx 24rpx;
		box-sizing: border-box;
		color: #2C405A;
	}			   
	.sendBox{		/*发送容器*/
		height: 100rpx;
		line-height: 100rpx;
		padding:0rpx 24rpx;
	}
	.sliderBox{		/*表情容器默认隐藏*/
		height: 0rpx;
		position: relative;
		z-index: -2;
		transition: all 0.3s;
	}
	.sliderBoxAc{  /*表情容器显示*/
		height: 300rpx !important;
		z-index: 1 !important;
	}
	.nothing{	   /*跟表情容器高度一直，但是不需要显示再页面，js需要拿这个高度*/
		height: 300rpx !important;
		position: absolute;
		z-index: -2;
	}
	/* 底部样式结束 */
	
	
	
	/* 表情轮播 */
	.swiper{
		height: 100% !important;
	}
	.swiperItem{
		background: white;
		padding:24rpx;
		box-sizing: border-box;
	}
	.swiperItem>view{
		width: 12.5%;
		height:84rpx;					/*300-48 再除以3*/
		box-sizing: border-box;
		display: flex;
		justify-content: center;
		align-items: center;
		float: left;
	}
	.swiperItem>view image{
		width:36rpx;
		height:36rpx;
	}
</style>