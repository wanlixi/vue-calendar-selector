<!-- 2017/11/21 version2.0 Create By Wanlixin -->
<template>
	<div class="calendar">
		<!-- 星期 -->
		<div class="weeks">
			<ul class="week">
				<li v-for="(item, index) in weeks" :key="index">{{item}}</li>
			</ul>
		</div>
		<div class="controls">
			<i class="last" @click="lastMonth"><</i>
			<span>{{controlsYear}}年{{controlsMonth | filterDate}}月</span>
			<i class="next" @click="nextMonth">></i>
		</div>
		<div class="datepicker"
			@touchstart="datepickerStart($event)"
			@touchmove="datepickerMove($event)"
			@touchend="datepickerEnd($event)">
			<ul>
				<!-- 显示当前月的1号不为周一时，前面空缺的上个月的日子 -->
				<li class="day before_curr_date_days"
					v-for="(item, index) in beforeDays"
					:key="index">{{lastMonthDays - currMonthIsWeek + index + 1}}</li>
				<!-- 当前月
					 selected_day 已经预约的类名
					 before_curr_date_days 上个月和下个月的日期
				-->
				<li 
					v-for="(item, index) in curMonthDays"
					:class="{
						'selected_day': index + 1 == controlsDay, 
						'appointed_day': item.appointed,
						'before_curr_date_days': index < currDay && currYear == controlsYear && currMonth == controlsMonth
					}"
					@click="SELECT_DATE(index)">
					<span class="day">{{index + 1}}</span>
					<span class="appoint appointed" v-if="item.appointed">已预约</span>
					<span
						v-else-if="index + 1 == controlsDay"
						:class="[item.classColor, 'appoint']">预约</span>
				</li>
				<!-- 显示当前月的最后一天没有排到第35个格时，剩余的下月的日子 +100 防止index冲突 -->
				<li class="day before_curr_date_days" 
					v-for="(item, index) in afterDays"
					:key="index + 100">{{index + 1}}</li>
				</ul>
			</div>
		</div>
	</template>
	<script>
	export default {
		data () {
			return {
				weeks: ['日','一','二','三','四','五','六'],
				beforeDays: 0, // 上月还剩几天
				lastMonthDays: 0, // 上月共多少天
				// curMonthDays: 0, // 本月一共几天
				curMonthDays: [], 
				afterDays: 0, // 下月还剩几天
				currMonthIsWeek: 0, // 本月一号是周几
				currYear: new Date().getFullYear(), // 今天是几年
				currMonth: new Date().getMonth() + 1, // 今天是几月
				currDay: new Date().getDate(), // 今天是几号
				controlsYear: new Date().getFullYear(), // 日期控制栏是多少年份
				controlsMonth: new Date().getMonth() + 1, // 日期控制栏是多少月份
				controlsDay: new Date().getDate(), // 日期控制栏是多少月份
				totalRow: 5, // 当前月份占几行
				pickerStartX: 0,
				pickerStartY: 0,
				pickerEndX: 0,
				pickerEndY: 0,
				swiperLimitX: 100,
				swiperLimitY: 50,
				submitMonthFormat: ''
			}
		},
		mounted () {
			// 初始化日期选择器
			this.initDatePicker(this.controlsMonth);
		},
		methods: {
			initDatePicker (month, yeah = 2017) {
				this.curMonthDays = [];
				let curMonthDaysLen = new Date(yeah, month, 0).getDate();
				for(let i = 0; i < curMonthDaysLen; i++) {
					this.curMonthDays.push({day: i + 1, appointed: false});
				}
				this.lastMonthDays = new Date(yeah, month - 1, 0).getDate();
				this.submitMonthFormat = month < 10 ? `0${month}` : month;
				this.currMonthIsWeek = new Date(`${yeah}-${this.submitMonthFormat}-01`).getDay();
				this.beforeDays = this.currMonthIsWeek;
				this.totalRow = this.beforeDays + this.curMonthDays.length > this.totalRow * 7 ? 6 : 5;
				this.afterDays = this.totalRow * 7 - this.currMonthIsWeek - this.curMonthDays.length;
				this.getByCusIdAndDate(`${this.currYear}-${this.submitMonthFormat}`);
			},
			lastMonth () { // 点击上一月
				this.controlsMonth --;
				// 控制不可以选择过去的日子
				this.controlsYear == this.currYear && this.controlsMonth < this.currMonth && (this.controlsMonth = this.currMonth)
				if (this.controlsMonth < 1) {
					this.controlsMonth = 12;
					this.controlsYear --;
				}
				this.initDatePicker(this.controlsMonth, this.controlsYear);
			},
			nextMonth () { // 点击下一月
				this.controlsMonth ++;
				if (this.controlsMonth > 12) {
					this.controlsMonth = 1;
					this.controlsYear ++;
				}
				this.initDatePicker(this.controlsMonth, this.controlsYear);
			},
			datepickerStart (ev) {
				this.pickerStartX = ev.touches[0].pageX;
				this.pickerStartY = ev.touches[0].pageY;
			},
			datepickerMove (ev) {
				this.pickerEndX = ev.touches[0].pageX;
				this.pickerEndY = ev.touches[0].pageY;
			},
			datepickerEnd () { // 左、右滑动出发上、下月
				this.pickerEndX - this.pickerStartX > this.swiperLimitX && Math.abs(this.pickerEndY - this.pickerStartY) < this.swiperLimitY && this.lastMonth();
				this.pickerStartX - this.pickerEndX > this.swiperLimitX && Math.abs(this.pickerEndY - this.pickerStartY) < this.swiperLimitY && this.nextMonth();
				this.pickerStartX = this.pickerStartY = this.pickerEndX = this.pickerEndY = 0;
			},
			SELECT_DATE (index) { // 选择某一天的日期，传递给了父组件
				this.controlsDay = index + 1;
				this.$store.commit('SELECT_DATE', {
					year: this.controlsYear,
					month: this.controlsMonth,
					day: index + 1
				})
			},
			getByCusIdAndDate (date) { // 获取后台已预约数据开始‘描点’
				this.$fetch.getByCusIdAndDate(date).then(res =>{
					res.data.map(item => {
						let arrDate = new Date(item.ARRIVE_DATE).getDate();
						for(let i = 0; i < item.LIVE_DAYS; i++) {
							this.curMonthDays[arrDate - 1 + i].appointed = true;
						}
					})
				})
			}
		}
	}
	</script>
	<style lang="stylus" scoped>
	@import '../assets/css/common/calendar.styl'
	</style>