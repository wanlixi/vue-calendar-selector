# calendar-selector
<!-- 2017/11/18 Create By Wanlixin -->
```
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
      <span>{{controlsYear}}年{{controlsMonth}}月</span>
      <i class="next" @click="nextMonth">></i>
		</div>
		<div class="datepicker">
			<ul>
				<!-- 显示当前月的1号不为周一时，前面空缺的上个月的日子 -->
				<li class="day" 
					:class="{'before_curr_date_days': currYear == controlsYear && currMonth == controlsMonth}" 
					v-for="(item, index) in beforeDays">{{lastMonthDays - currMonthIsWeek + (index + 1)}}</li>
				<!-- 当前月
						 appointed 已经预约的类名
						 before_curr_date_days 上个月和下个月的日期
				 -->
				<li 
					v-for="(item, index) in curMonthDays"
					:class="{'appointed': index+1 == currDay && currMonth == controlsMonth, 'before_curr_date_days': index < currDay && currYear == controlsYear && currMonth == controlsMonth}">
					<span class="day">{{index+1}}</span>
					<span
						v-if="index+1 == currDay && currMonth == controlsMonth"
						:class="[item.classColor, 'appoint']">预约</span>
				</li>
				<!-- 显示当前月的最后一天没有排到第35个格时，剩余的下月的日子 -->
				<li class="day before_curr_date_days" 
					v-for="(item, index) in afterDays">{{index+1}}</li>
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
				curMonthDays: 0, // 本月一共几天
				afterDays: 0, // 下月还剩几天
				currMonthIsWeek: 0, // 本月一号是周几
				// currYear: new Date().getFullYear(), // 今天是几年
				// currMonth: new Date().getMonth() + 1, // 今天是几月
				// currDay: new Date().getDate(), // 今天是几号
				controlsYear: new Date().getFullYear(), // 日期控制栏是多少年份
				controlsMonth: new Date().getMonth() + 1, // 日期控制栏是多少月份
				totalRow: 5, // 当前月份占几行
			}
		},
		props: {
			currYear: {
				type: Number,
				default: new Date().getFullYear(),
			},
			currMonth: {
				type: Number,
				default: new Date().getFullYear(),
			},
			currDay: {
				type: Number,
				default: new Date().getFullYear(),
			},
		},
		mounted () {
			// 初始化日期选择器
			this.initDatePicker(this.controlsMonth);
		},
		methods: {
			initDatePicker (month, yeah = 2017) {
				this.curMonthDays = new Date(yeah, month, 0).getDate();
				this.lastMonthDays = new Date(yeah, month - 1, 0).getDate();
				this.currMonthIsWeek = new Date(yeah + '-' + month + '-01').getDay();
				this.beforeDays = this.currMonthIsWeek;
				this.totalRow = this.beforeDays + this.curMonthDays > this.totalRow * 7 ? 6 : 5;
				this.afterDays = this.totalRow * 7 - this.currMonthIsWeek - this.curMonthDays;
			},
			lastMonth () { // 点击上一月
				this.controlsMonth --;
				// 控制不可以选择过去的日子
				if (this.controlsYear == this.currYear && this.controlsMonth < this.currMonth) {
					this.controlsMonth = this.currMonth;
				}
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
		}
	}
</script>
<style lang="stylus" scoped>
@import './var.styl'

$boxWidth = 1rem; // 50px 单元格的宽度
$invalidDateColor = #ccc // 无效日期（过去的日子）的表示颜色
$validDateColor = #222 // 有效日期（过去的日子）的表示颜色

// 星期栏
.weeks
	background #fafafa
.week
	display flex
	width 7rem; // 350px
	height .96rem; // 48px
	margin auto
.week li
	width $boxWidth
	line-height .96rem; // 48px
	text-align center
.week li:nth-child(1),.week li:nth-last-child(1)
	color $commonYellow

// 年月日前进、后退日期控制栏
.controls
	display flex
	justify-content space-around
	align-items center
	height .96rem; // 48px
	padding 0 1.6rem // 80px
	border-bottom 1px solid #f4f4f4

.datepicker ul
	width $boxWidth * 7
	margin auto
	overflow hidden
	li 
		float left
		display flex
		flex-flow column nowrap
		justify-content center
		align-items center
		width $boxWidth
		height $boxWidth
		color #222
	.before_curr_date_days
		color $invalidDateColor
		
.datepicker ul .appointed
	color #fff
	background $commonYellow

.appoint 
	color #fff

	
.day 
	font-size .4rem; // 20px

.appoint
	font-size .24rem; // 12px
</style>
```
