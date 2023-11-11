<template>
	<view class="container">
		<view class="indate">入住时间：{{ checkInDate }}</view>
		<view class="outdate">离开时间：{{ checkOutDate }}</view>
		<view class="">共{{ day }}晚</view>
		<navigator url="/pages/calendar/index">
			<button>日期选择</button>
		</navigator>
	</view>
</template>

<script>
	import Moment from "../../utils/moment.js"
	export default {
		data() {
			return {
				checkInDate: '',
				checkOutDate: '',
				day: '',
			}
		},
		onLoad() {
			this.checkInDate = Moment(new Date()).format("YYYY-MM-DD");
			this.checkOutDate = Moment(new Date()).add(1, "day").format("YYYY-MM-DD");
			uni.setStorageSync("CHECKED_DATE", { checkInDate: this.checkInDate, checkOutDate: this.checkOutDate });
		},
		onShow() {
			let date = uni.getStorageSync("CHECKED_DATE");
			this.checkInDate = date.checkInDate;
			this.checkOutDate = date.checkOutDate;
			this.day = Moment(date.checkOutDate).differ(date.checkInDate);
		},
	}
</script>

<style>
	.container {
		padding: 20upx;
		margin-top: 100upx;
		display: flex;
		flex-direction: column;
		align-items: center;
	}
	
	.container view {
		margin-bottom: 20upx;
	}
</style>
