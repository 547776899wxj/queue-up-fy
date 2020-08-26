<template>
	<view class="content" @longpress="open" @click="open">
		<image class="bg" src="/static/logo.png"></image>
		<view class="header-time">
			<text>{{ dateText.year }}年{{ dateText.month }}月{{ dateText.date }}日</text>
			<view>
				<text style="padding-right: 5px;">{{ dateText.day }}</text>
				<text>{{ dateText.time }}</text>
			</view>
		</view>
		<view class="info">
			<view class="info-patient" v-for="(item,index) in data" :key="index">
				<view :class="{first:(index==0)}">{{item.number}}</view>
				<view :class="{first:(index==0)}">{{item.name}}</view>
				<view :class="{first:(index==0)}">{{item.windowNumber}}</view>
			</view>
		</view>
		<uni-popup ref="popup" type="center" :maskClick="false">
			<view class="popup">
				<view class="popup-header">
					设置
				</view>
				<view>
					<!-- <view class="uni-form-item " style="padding-bottom: 0;">
						<view class="popup-title">窗口编号：</view>
						<input class="uni-input" v-model="cliniqueCode"  focus placeholder="请输入窗口编号" />
					</view> -->
					<view class="uni-form-item ">
						<view class="popup-title">队列类型：</view>
						<input class="uni-input" v-model="iType"  placeholder="请输入队列类型" />
					</view>
					<view class="uni-form-item " >
						<button class="popup-btn" @click="close">取消</button>
						<button class="popup-btn" @click="confirm">确定</button>
					</view>
				</view>
			</view>
		</uni-popup>
	</view>
</template>

<script>
import uniPopup from '@/components/uni-popup/uni-popup.vue'
var FvvUniTTS = uni.requireNativePlugin('Fvv-UniTTS')
export default {
	data() {
		return {
			dateText: {
				year: '',
				month: '',
				date: '',
				day: '',
				time: ''
			},
			weekday: [],
			data:[
				// {
				// 	number:"0720",
				// 	name:'张*鑫',
				// 	windowNumber:'窗口3',
				// },
				// {number:"0721",name:'张*鑫',windowNumber:'窗口3'},
				// {number:"0723",name:'张*',windowNumber:'窗口3'}
			],
			cliniqueCode:'',
			iType:'',
			popupShow:false,
			seqNumber:'',
			test:'测试',
			testNubmer:0,
		};
	},
	onLoad() {
		// this.cliniqueCode = uni.getStorageSync('cliniqueCode')||'';
		this.iType = uni.getStorageSync('iType')||'';
		FvvUniTTS.init((callback) => {
			// FvvUniTTS.setEngine("com.baidu.duersdk.opensdk");
		});
		let date = new Date();
		this.weekday = new Array(7);
		this.weekday[0] = '星期日';
		this.weekday[1] = '星期一';
		this.weekday[2] = '星期二';
		this.weekday[3] = '星期三';
		this.weekday[4] = '星期四';
		this.weekday[5] = '星期五';
		this.weekday[6] = '星期六';
		this.newDate();
		setTimeout(() => {
			this.newDate();
			setInterval(() => {
				this.newDate();
			}, 60000);
		}, date.getSeconds() * 1000);
		if(this.iType){
			this.init();
		}
	},
	methods: {
		//当前时间
		newDate() {
			let date = new Date();
			this.dateText = {
				year: date.getFullYear(),
				month: date.getMonth() + 1,
				date: date.getDate(),
				day: this.weekday[date.getDay()],
				time: date.getHours() + ':' + (date.getMinutes() < 10 ? '0' + date.getMinutes() : date.getMinutes())
			};
		},
		// 打开设置
		open(){
			this.$refs.popup.open();
			this.popupShow = true;
		},
		// 关闭设置
		close(){
			this.$refs.popup.close();
			this.popupShow = false;
			
		},
		//确定设置
		confirm(){
			// if(this.cliniqueCode===''){
			// 	uni.showToast({
			// 		title:'请输入窗口编号',
			// 		icon:'none'
			// 	})
			// 	return
			// }
			if(this.iType===''){
				uni.showToast({
					title:'请输入队列类型',
					icon:'none'
				})
				return
			}
			
			uni.showLoading({
				title: '加载中',
			});
			// uni.setStorageSync('cliniqueCode',this.cliniqueCode);
			uni.setStorageSync('iType',this.iType);
			this.popupShow = false;
			this.data = [];
			this.init();
			this.$refs.popup.close();
			uni.hideLoading();
		},
		// 初始化数据
		init(){
			if(this.popupShow){
				return false;
			}
			// 测试使用
			// let data = {seq_number:"adadasdsda",patient_name:"王勇勇",clinique_name:"窗口10",doctor_seq:"123"};
			// data.doctor_seq = data.doctor_seq + this.testNubmer++
			
			
			uni.request({
			    url: 'http://198.100.100.36:8018/Queue/Get_Queue', 
			    // url: 'http://192.168.0.159:8018/Queue/Get_Queue', 
				data:{
					// Clinique_Code:this.cliniqueCode,
					iType :this.iType ,
				},
				timeout:3000,
			    success: (res) => {
					let data = res.data.Data;
					if(!data.patient_name){
						setTimeout(() => {
							this.init()
						}, 3000);
						return;
					}
					let name = data.patient_name;
					if(name.length==2){
					    name = '*'+name.slice(1,name.length)
					}else if(name.length>2){
						name = name.slice(0,1) + '*' + name.slice(name.length-1,name.length)
					}
					let booleanSame = false;
					// if(this.data.length>0){
					// 	this.data.forEach(item =>{
					// 		if(item.number == data.doctor_seq){
					// 			booleanSame = true;
					// 		}
					// 	})
					// }
					if(!booleanSame){
						this.seqNumber = data.seq_number;
						let dataMap = {
							number:data.doctor_seq,
							name:name,
							windowNumber:data.clinique_name,
						}
						// 请 票号  患者名 到 窗口名
						let number = this.chineseNumeral(dataMap.number+'');
						// let windowNumber =  this.chineseNumeral(dataMap.windowNumber+'');
						let windowNumber =  dataMap.windowNumber;
						let speakText = `请,${number}号,${data.patient_name}到,${windowNumber}`;
						FvvUniTTS.init((callback) => {
							FvvUniTTS.speak({
								text:speakText
							});
						});
						this.onDone(speakText);
						if(this.data.length<3){
							this.data = this.data.concat(dataMap)
						}else{
							this.data[2] = dataMap; 
							this.$forceUpdate();
						}
					}else{
						setTimeout(() => {
						this.init()
						}, 3000);
					}
			    },
				fail:(res) => {
					uni.showToast({
						title:'请求失败',
						icon:'none'
					})
				}
			});
		},
		// 播放完执行
		onDone(data){
			let date = 4300;
			if(data.length>12){
				date = date + ((data.length - 12)*300 ) 
			}
			setTimeout(() => {
				uni.request({
				    url: 'http://198.100.100.36:8018/Queue/Set_Queue', 
				    // url: 'http://192.168.0.159:8018/Queue/Set_Queue', 
					data:{
						Seq_Number:this.seqNumber,
					},
					timeout:3000,
				    success: (res) => {
						this.init();
				    },
				});
			}, date);
			
		},
		//转大写
		chineseNumeral(data){
			let tmpnewchar = "" ;
				for(let char of data){
					switch (char) {
			            case "0":   tmpnewchar =  tmpnewchar + "零" ;break;
			            case "1":  tmpnewchar =  tmpnewchar + "一" ; break;
			            case "2":  tmpnewchar =  tmpnewchar + "二" ; break;
			            case "3":  tmpnewchar =  tmpnewchar + "三" ; break;
			            case "4":  tmpnewchar =  tmpnewchar + "四" ; break;
			            case "5":  tmpnewchar =  tmpnewchar + "五" ; break;
			            case "6":  tmpnewchar =  tmpnewchar + "六" ; break;
			            case "7":  tmpnewchar =  tmpnewchar + "七" ; break;
			            case "8":  tmpnewchar =  tmpnewchar + "八" ; break;
			            case "9":  tmpnewchar =  tmpnewchar + "九" ; break;
						default: tmpnewchar = tmpnewchar + char;
			        }
			}
			return tmpnewchar;
		}
	}
};
</script>

<style>
.content {
	position: relative;
}
.bg {
	position: absolute;
	height: 720px;
	width: 1280px;
	z-index: -1;
}
.header-time {
	display: flex;
	justify-content: center;
	flex-direction: column;
	align-items: flex-end;
	padding-right: 20px;
	height: 93px;
}
.header-time text {
	color: #fff;
	font-size: 23px;
	text-align: center;
}
.header-time view {
	line-height: 0.3;
	width: 164px;
	text-align: center;
}
.info{
	padding-top: 81px;
	padding-left: 27px;
}
.info-patient {
	display: flex;
	height: 162px;
	
}
.info-patient view {
	width: 409px;
	font-size: 65px;
	color: #fff;
	display: flex;
	align-items: center;
	justify-content: center;
}
.info-patient .first{
	color: rgb(255,254,94);
}
.popup-btn{
		font-size: 30px;
		color: #fff;
		padding-left: 40px;
		padding-right: 40px;
		background-color: rgb(68,114,196);
		margin-left: 40px;
		margin-right: 40px;
	}
	.popup{
		background-color: #fff;
		width: 600px;
		font-size: 40px;
		z-index: 100;
	}
	.popup-header{
		background-color: rgb(68,114,196);
		text-align: center;
		padding: 10px 0 ;
	}
	.uni-form-item{
		display: flex;
		align-items: center;
		padding: 40px;
		justify-content: center;
	}
	.popup-title{
		font-size: 30px;
	}
	.uni-input{
		font-size: 25px;
		border: 1px solid;
		padding: 20px 30px;
	}
</style>
