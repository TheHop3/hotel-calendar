<template>
	<view class="container">
		<view class="calendar-header">
			<text v-for="item in weekStr" :key="item" :style="{ width: (screenWidth / 7 - 10) + 'px' }">{{item}}</text>
		</view>
		 <view class="placeholder"></view>
		 <view class="calendar-content">
			 <view class="calendar-center" v-for="(item, index) in dateList" :key="index">
				 <view class="center-header">{{item.year}}年{{item.month}}月</view>
				 <view class="center-content">
					 <view class="center-day" 
						v-for="(temp, idx) in item.days" :key="idx" 
						:class="[temp.day <= 0 ? 'bgwhite' : temp.class]"
						:style="{ width: (screenWidth / 7 - 10) + 'px', height: screenWidth / 7 + 'px' }"
						@click="opPressDate(item.year, item.month, temp.day)"
					>
						 <view class="item">
							 <text style="font-size: 28rpx">{{ temp.day > 0 ? temp.day : '' }}</text>
							 <text style="font-size: 30rpx" v-if="temp.inday">入住</text>
							 <text style="font-size: 30rpx" v-if="temp.outday">离店</text>
						 </view>
					 </view>
				 </view>
			 </view>
		 </view>
	</view>
</template>

<script>
	import Moment from "../../utils/moment.js"
	export default {
		data() {
			return {
				DATE_LIST: [],
				DATE_YEAR: new Date().getFullYear(),
				DATE_MONTH: new Date().getMonth() + 1,
				DATE_DAY: new Date().getDate(),
				
				weekStr: ['日', '一', '二', '三', '四', '五', '六'],
				checkInDate: Moment(new Date()).format('YYYY-MM-DD'),
				checkOutDate: Moment(new Date()).add(1, 'day').format('YYYY-MM-DD'),
				isCheckInDate: false, //标记开始时间是否已经选择
				isCheckOutDate: false,   //标记结束时间是否已经选择
				maxMonth: 6,
				dateList: [],
				
				screenWidth: '',
			}
		},
		onLoad(options) {
			this.getSystemInfo();
			this.createDateList();
		},
		onShow() {
			this.selectDataLine();
		},
		methods: {
			createDateList() {
				let dateList = [];
				let now = new Date();
				/**
				 * 设置日期为 年-月-01，否则跨月时可能出现问题
				 * 例如：2023-01-31 为 now，月份直接 +1 ，则会直接跳到2023-03-03
				 * 原因是 2月份 没有 31号，推下去就变成了03-03
				 */
				now = new Date(now.getFullYear(), now.getMonth(), 1);
				for (let i = 0; i < this.maxMonth; i++) {
					let momentDate = Moment(now).add(this.maxMonth - (this.maxMonth - i), 'month').date;
					let year = momentDate.getFullYear();
					let month = momentDate.getMonth() + 1;
					
					let days = [];
					let totalDay = this.getTotalDayByMonth(year, month);
					let week = this.getWeek(year, month, 1);
					
					/**
					 * -week 是为了使当月的第一天的日期可以正确的显示到对应的周几上，比如 星期六（week = 6）;
					 * 则 当月的 1号 是从列的第七个位置开始渲染的，前面会占用 -5，-4，-3，-2，-1，0 的位置，从 1 开始渲染
					 */
					for (let i = -week + 1; i <= totalDay; i++) {
						let tempWeek = -1;
						if (i > 0) tempWeek = this.getWeek(year, month, i);
						let clazz = '';
						if (i < this.DATE_DAY && year == this.DATE_YEAR && month == this.DATE_MONTH) 
						// 此前日期不可用
							clazz = 'unavailable ' + clazz;
						else
							clazz = '' + clazz;
						
						days.push({ day: i, class: clazz });
					}
					
					let dateItem = {
						id: year + '-' + month,
						year: year,
						month: month,
						days: days
					}
					
					dateList.push(dateItem);
				}
				this.dateList = dateList;
				this.DATE_LIST = dateList;
			},
			getSystemInfo() {
				this.screenWidth = uni.getSystemInfoSync().windowWidth;
			},
			/**
			 * 获取月的总天数
			 */
			getTotalDayByMonth(year, month) {
				month = parseInt(month, 10);
				let d = new Date(year, month, 0);
				return d.getDate();
			},
			/**
			 * 获取月的第一天是周几
			 */
			getWeek(year, month, day) {
				let d = new Date(year, month - 1, day);
				return d.getDay();
			},
			/**
			 * 选择入住和离店的时间段
			 */
			selectDataLine() {
				let dateList = this.dateList;
				let { checkInDate, checkOutDate } = uni.getStorageSync("CHECKED_DATE");
				// 选择入住的 id
				let currentInid = checkInDate.substr(0, 4) + '-' + (checkInDate.substr(5, 2) < 10 ? checkInDate.substr(6, 1) : checkInDate.substr(5, 2));
				
				// 选择离店的 id
				let currentOutid = checkOutDate.substr(0, 4) + '-' + (checkOutDate.substr(5, 2) < 10 ? checkOutDate.substr(6, 1) : checkOutDate.substr(5, 2));
				
				// 选择入住天的 id
				let dayIn = checkInDate.substr(8, 2) >= 10 ? checkInDate.substr(8, 2) : checkInDate.substr(9, 1);
				
				// 选择离店天的 id
				let dayOut = checkOutDate.substr(8, 2) >= 10 ? checkOutDate.substr(8, 2) : checkOutDate.substr(9, 1);
				
				// 选择入住月的 id
				let monthIn = checkInDate.substr(5, 2) >= 10 ? checkInDate.substr(5, 2) : checkInDate.substr(6, 1);
				
				// 选择离店月的 id
				let monthOut = checkOutDate.substr(5, 2) >= 10 ? checkOutDate.substr(5, 2) : checkOutDate.substr(6, 1);
				
				if (currentInid == currentOutid) {
					// 入住和离店是同月的情况
					for (let i = 0; i < dateList.length; i++) {
						if (dateList[i].id == currentInid) {
							let days = dateList[i].days;
							for (let j = 0; j < days.length; j++) {
								if (days[j].day >= dayIn && days[j].day <= dayOut) {
									days[j].class = days[j].class + ' bgitem';
								}
								if (days[j].day == dayIn) {
									days[j].class = days[j].class + ' active';
									days[j].inday = true;
								}
								if (days[j].day == dayOut) {
									days[j].class = days[j].class + ' active';
									days[j].outday = true;
								}
							}
						}
					}
				} else {
					// 跨月情况
					for (let i = 0; i < dateList.length; i++) {
						if (dateList[i].month == monthIn) {
							// 入住的开始月份
							let days = dateList[i].days;
							for (let j = 0; j < days.length; j++) {
								if (days[j].day >= dayIn) {
									days[j].class = days[j].class + ' bgitem';
								}
								if (days[j].day == dayIn) {
									days[j].class = days[j].class + ' active';
									days[j].inday = true;
								}
							}
						} else {
							// 入住跨月月份
							if (Number(dateList[i].month) < Number(monthOut) || Number(dateList[i].month) > Number(monthIn)) {
								// 离店中间的月份
								let days = dateList[i].days;
								for (let j = 0; j < days.length; j++) {
									days[j].class = days[j].class + ' bgitem';
								}
							} else if (Number(dateList[i].month) == Number(monthOut)) {
								// 离店最后的月份
								let days = dateList[i].days;
								for (let j = 0; j < days.length; j++) {
									if (days[j].day <= dayOut) {
										days[j].class = days[j].class + ' bgitem';
									}
									if (days[j].day == dayOut) {
										days[j].class = days[j].class + ' active';
										days[j].outday = true;
									}
								}
							}
						}
					}
				}
				this.dateList = dateList;
			},
			/**
			 * 点击日期事件
			 */
			opPressDate(year, month, day) {
				// 当前选择的日期为同一个月 并 小于今天，或者点击了空白处（day < 0）,不执行任何操作
				if ((day < this.DATE_DAY && month == this.DATE_MONTH) || day <= 0) return;
				
				let tempMonth = month;
				let tempDay = day;
				
				if (month < 10) tempMonth = '0' + month;
				if (day < 10) tempDay = '0' + day;
				
				let date = year + '-' + tempMonth + '-' + tempDay;
				
				// 如果点击选择的日期A 小于 入住时间 ，则重新渲染入住时间 为 A
				if (this.isCheckInDate && Moment(date).before(this.checkInDate) || this.checkInDate == date) {
					this.isCheckInDate = false;
					this.isCheckOutDate = false;
					this.dateList = this.DATE_LIST.concat();
				}
				
				if (!this.isCheckInDate) {
					this.checkInDate = date;
					this.isCheckInDate = true;
					this.dateList = this.DATE_LIST.concat();
				} else if (!this.isCheckOutDate) {
					this.checkOutDate = date;
					this.isCheckOutDate = true;
					
					// 设置缓存，返回之前页面是，在onShow中获取
					uni.setStorageSync("CHECKED_DATE", { checkInDate: this.checkInDate, checkOutDate: this.checkOutDate });
					uni.navigateBack({
						delta: 1,
					});
				}
				
				this.renderPressStyle(year, month, day);
			},
			/**
			 * 重新渲染
			 */
			renderPressStyle(year, month, day) {
				// 重新点击时数据初始化
				this.createDateList();
				
				// 渲染
				let dateList = this.dateList;
				for (let i = 0; i < dateList.length; i++) {
					let dateItem = dateList[i];
					let id = dateItem.id;
					if (id === year + '-' + month) {
						let days = dateItem.days;
						for (let j = 0; j < days.length; j++) {
							let tempDay = days[j].day;
							if (tempDay == day) {
								days[j].class = days[j].class + ' active';
								days[j].inday = true;
								break;
							}
						}
						break;
					}
				}
				
				this.dateList = dateList;
			}
		}
	}
</script>

<style>
	.calendar-header {
		width: 100%;
		display: flex;
		align-items: center;
		justify-content: space-around;
		padding: 16upx 0;
		margin-bottom: 16upx;
		position: fixed;
		top: 0;
		background: #fff;
	}
	
	.calendar-header text {
		/* font-size: 24upx; */
		color: #666;
		margin: 5upx;
		display: flex;
		justify-content: center;
		align-items: center;
	}
	
	.placeholder {
		margin-top: 60upx;
	}
	
	.calendar-content {
		width: 100%;
		background: #fff;
		padding: 16upx 0;
	}
	
	.calendar-center {
		margin-top: 16upx;
	}
	
	.center-header {
		font-size: 32upx;
		color: #333;
		text-align: center;
	}
	
	.center-content {
		margin-top: 20upx;
		display: flex;
		flex-wrap: wrap;
		flex-direction: row;
	}
	
	.bgitem {
		background: #d8ecf9;
	}
	
	.active {
		background: #D4AD6D;
		color: #fff;
		border-radius: 8upx;
	}
	
	.center-day {
		padding: 10upx;
		display: flex;
		justify-content: center;
		align-items: center;
		flex-direction: column;
	}
	
	.item {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		font-size: 35upx;
	}
	
	.bgwhite{
	  background-color: #fff;
	}
	
	.unavailable {
		color: #aaa;
	}
</style>