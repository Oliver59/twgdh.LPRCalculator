<template>
	<view class="page">
	<!-- 	<view class="title">
			<view class="status_bar"></view>
			<view class="status_text">还款详情</view>
		</view> -->
		<view class="content">
			<view class="header">
				<view class="option">
					<view class="item" :class="{ action: repaymentType == 1 }" v-on:click="bindRepaymentTypeChange(1)">等额本息</view>
					<view class="item" :class="{ action: repaymentType == 2 }" v-on:click="bindRepaymentTypeChange(2)">等额本金</view>
				</view>
				<view class="mainPrice">
					<view class="item">
						<text class="text">总还款(万)</text>
						<text class="value">{{ toMoney(this.calcuteResult.totalPrice / 10000) }}</text>
					</view>
					<view class="item">
						<text class="text">总利息(万)</text>
						<text class="value">{{ toMoney(this.calcuteResult.totalLixi / 10000) }}</text>
					</view>
					<view class="item">
						<text class="text">总贷款(万)</text>
						<text class="value">{{ toMoney(this.calcuteResult.totalDknum) }}</text>
					</view>
					<view class="item">
						<text class="text">贷款月数</text>
						<text class="value">{{ this.getMonths() }}</text>
					</view>
				</view>
			</view>
			<view class="main">
				<view class="rowHead">
					<view class="item">月份</view>
					<view class="item">月供本金</view>
					<view class="item">月供利息</view>
					<view class="item">剩余</view>
				</view>
				<view class="rowBody">
					<view class="yearRegion" v-for="(item, index) in calcuteResult.yearArr">
						<view class="rowYearText">第{{ index + 1 }}年</view>
						<view class="monthRegion">
							<view class="rowMoth" v-for="(month_item, month_index) in item.monthArr">
								<view class="item">{{ month_index + 1 }}月</view>
								<view class="item">{{ toMoney(month_item.yuebenjin) }}</view>
								<view class="item">{{ toMoney(month_item.yuelixi) }}</view>
								<view class="item">{{ toMoney(month_item.leftFund) }}</view>
							</view>
						</view>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
import calcute from '@/pages/index/calculate.js';
export default {
	data() {
		return {
			repaymentType: 1, //1等额本息 2等额本金
			year: 0,
			sdnum: 0,
			gjjnum: 0,
			sdlilv: 0,
			gjjlilv: 0,
			loansTypeIndex: 0,
			calcuteResult: null
		};
	},
	onLoad: function(option) {
		//option为object类型，会序列化上个页面传递的参数
		this.repaymentType = option.repaymentType;
		this.year = option.year;
		this.sdnum = option.sdnum;
		this.gjjnum = option.gjjnum;
		this.sdlilv = option.sdlilv;
		this.gjjlilv = option.gjjlilv;
		this.loansTypeIndex = option.loansTypeIndex;

		this.calculateLoan();
	},
	methods: {
		bindRepaymentTypeChange: function(value) {
			if (this.repaymentType == value) return;
			this.repaymentType = value;
			this.calculateLoan();
		},
		calculateLoan: function() {
			let flag = false;
			switch (Number(this.loansTypeIndex)) {
				case 0:
					// 商业贷款
					this.calcuteResult = calcute.singleDk(this.repaymentType, this.sdnum, this.year, this.sdlilv);
					flag = true;
					break;
				case 1:
					// 公积金贷款
					this.calcuteResult = calcute.singleDk(this.repaymentType, this.gjjnum, this.year, this.gjjlilv);
					flag = true;
					break;
				case 2:
					// 组合贷款
					this.calcuteResult = calcute.zuhe(this.repaymentType, this.sdnum, this.gjjnum, this.year, this.year, this.sdlilv, this.gjjlilv);
					flag = true;
					break;
			}
			if (flag) {
				this.calcuteResult.yearArr = [];
				for (let i = 1; i <= this.year; i++) {
					this.calcuteResult.yearArr.push({
						yearIndex: i,
						monthArr: []
					});
				}
				console.log(this.calcuteResult);
				for (let i = 0; i < this.calcuteResult.mouthdataArray.length; i++) {
					let index = Math.floor(i / 12);
					this.calcuteResult.yearArr[index].monthArr.push(this.calcuteResult.mouthdataArray[i]);
				}
				console.log("this.calcuteResult",this.calcuteResult);
			}
		},
		toMoney: function(number) {
			var decimals = 2;
			var dec_point = '.';
			var thousands_sep = ',';
			/*
			 * 参数说明：
			 * number：要格式化的数字
			 * decimals：保留几位小数
			 * dec_point：小数点符号
			 * thousands_sep：千分位符号
			 * */
			number = (number + '').replace(/[^0-9+-Ee.]/g, '');
			var n = !isFinite(+number) ? 0 : +number,
				prec = !isFinite(+decimals) ? 0 : Math.abs(decimals),
				sep = typeof thousands_sep === 'undefined' ? ',' : thousands_sep,
				dec = typeof dec_point === 'undefined' ? '.' : dec_point,
				s = '',
				toFixedFix = function(n, prec) {
					var k = Math.pow(10, prec);
					return '' + Math.floor(n * k) / k;
				};
			s = (prec ? toFixedFix(n, prec) : '' + Math.floor(n)).split('.');
			var re = /(-?\d+)(\d{3})/;
			console.log(s);
			while (re.test(s[0])) {
				s[0] = s[0].replace(re, '$1' + sep + '$2');
			}

			if ((s[1] || '').length < prec) {
				s[1] = s[1] || '';
				s[1] += new Array(prec - s[1].length + 1).join('0');
			}
			return s.join(dec);
		},
		getMonths:function(){
			return this.calcuteResult.mouthdataArray.length;
		}
	}
};
</script>

<style lang="less">
@import 'detail.less';
</style>
